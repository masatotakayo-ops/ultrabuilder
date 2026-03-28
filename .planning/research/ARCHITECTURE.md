# Architecture Patterns

**Domain:** Autonomous Software Factory / AI-Powered IDE Desktop App
**Researched:** 2026-03-28

## Recommended Architecture

Ultrabuilder is a layered desktop application with clear separation between the Tauri shell, the React UI, backend services (sidecars), and agent execution environments (Docker containers + git worktrees). The architecture follows a **hub-and-spoke** model: the Tauri Rust core is the hub that manages all process lifecycles, state, and IPC routing, while the React frontend, Python/Go sidecars, and Docker containers are spokes that communicate exclusively through the hub.

```
+------------------------------------------------------------------+
|                        Tauri Desktop Shell                        |
|  +--------------------+    +----------------------------------+  |
|  |    Rust Core        |    |       React + TypeScript UI      |  |
|  |                    |    |                                  |  |
|  |  - State Manager   |<-->|  - Studio Views                  |  |
|  |  - IPC Router      |    |  - Agent Live Feed               |  |
|  |  - Sidecar Manager |    |  - File Explorer                 |  |
|  |  - Docker Manager  |    |  - Terminal Emulator             |  |
|  |  - Git Manager     |    |  - Zustand State                 |  |
|  +--------+-----------+    +----------------------------------+  |
|           |                                                      |
+-----------|------------------------------------------------------+
            |
   +--------+--------+------------------+
   |                 |                  |
   v                 v                  v
+--------+    +-----------+    +---------------+
|DeerFlow|    |   Scion   |    | Docker Engine |
|(Python)|    |   (Go)    |    |               |
|FastAPI |    | gRPC/HTTP |    | Agent Sandboxes|
|sidecar |    | sidecar   |    | per worktree  |
+--------+    +-----------+    +---------------+
```

### Component Boundaries

| Component | Responsibility | Communicates With | Protocol |
|-----------|---------------|-------------------|----------|
| **Tauri Rust Core** | Process lifecycle, state management, IPC routing, security enforcement | All components | Native Rust APIs |
| **React Frontend** | UI rendering, user interaction, studio wizards, agent output display | Rust Core only | Tauri Commands + Channels + Events |
| **Sidecar Manager** (Rust) | Spawn/monitor/restart/kill DeerFlow and Scion processes | DeerFlow, Scion binaries | Tauri Shell Plugin (`@tauri-apps/plugin-shell`) |
| **DeerFlow Sidecar** | Agent execution, sub-agent spawning, sandbox orchestration | Rust Core (via HTTP), Docker Engine | HTTP/REST (FastAPI), Docker SDK |
| **Scion Sidecar** | Container lifecycle, worktree management, resource isolation | Rust Core (via HTTP), Docker Engine | HTTP/REST or gRPC |
| **Docker Manager** (Rust) | Container CRUD, image management, network setup | Docker Engine | Bollard crate (Rust Docker SDK) |
| **Git Manager** (Rust) | Worktree creation/cleanup, branch management, commit orchestration | Local git repos | git2 crate (libgit2 bindings) |
| **Flow Engine** (TS package) | Unified adapter over Ruflo + DeerFlow + Scion + EverMemOS | All 4 subsystems | TypeScript imports (Ruflo, EverMemOS) + HTTP clients (DeerFlow, Scion) |
| **Ruflo** (TS package) | Intelligence routing, policy enforcement, learning loops | Flow Engine | Direct TS import |
| **EverMemOS** (TS package) | Memory cells, consolidation, recall | Flow Engine | Direct TS import |
| **SQLite** | Local persistence (projects, agent history, settings) | Rust Core | tauri-plugin-sql or rusqlite |
| **Hono API** | Backend API for cloud sync, team features | React Frontend, Postgres | HTTP/REST |

### Data Flow

**Golden Path: User Intent to Working Code**

```
1. User defines product (React UI - Studio I)
        |
        v  [Tauri Command]
2. Rust Core receives PRD, persists to SQLite
        |
        v  [Flow Engine call via TS]
3. Ruflo plans task graph (intelligence routing)
        |
        v  [HTTP to sidecar]
4. DeerFlow receives execution plan
        |
        v  [Docker API via Bollard]
5. Docker containers spawn per agent
   Each gets its own git worktree
        |
        v  [Tauri Channel - streaming]
6. Agent output streams back through:
   Container stdout -> Scion -> Rust Core -> Channel -> React UI
        |
        v  [git2 crate]
7. Each agent commits to its worktree branch
        |
        v  [git merge]
8. Architect agent orchestrates branch merges to main
```

**IPC Data Flow (Frontend <-> Backend)**

```
React UI  --[invoke("command")]-->  Rust Command Handler
React UI  <--[Channel stream]----  Rust (agent output, progress, logs)
React UI  <--[Event emit]--------  Rust (lifecycle events: sidecar up/down, build status)
React UI  --[Event listen/emit]->  Rust (user actions: cancel agent, pause build)
```

**Agent Output Streaming (Critical Path)**

```
Docker Container (agent process)
    |  stdout/stderr
    v
Scion sidecar (captures output)
    |  HTTP POST /stream or WebSocket
    v
Rust Core (receives, tags with agent ID + timestamp)
    |  Tauri Channel (ordered, fast)
    v
React Frontend (multiplexed display per agent)
```

## Key Architectural Decisions

### 1. Tauri IPC: Commands + Channels + Events (Three-Tier)

Use Tauri's three IPC mechanisms for their intended purposes:

| Mechanism | Use For | Direction | Performance |
|-----------|---------|-----------|-------------|
| **Commands** (`invoke`) | Request/response: CRUD, config, one-shot queries | Frontend -> Rust | Medium latency, JSON serialized |
| **Channels** (`Channel`) | Streaming: agent output, progress bars, logs | Rust -> Frontend | High throughput, ordered delivery, supports raw bytes |
| **Events** (`emit`/`listen`) | Lifecycle: sidecar status, build phase changes, errors | Bidirectional | Lower throughput, fire-and-forget |

**Rationale:** Tauri 2.0 explicitly states that Channels are optimized for streaming (used internally for download progress, child process output, WebSocket messages) while Events are not designed for high throughput. Agent output streaming MUST use Channels, not Events.

**Confidence:** HIGH - directly from Tauri 2.0 official documentation.

### 2. Sidecar Management: Rust-Side Lifecycle Controller

Do NOT manage DeerFlow/Scion processes from the frontend. The Rust core owns all process lifecycles:

```rust
// Conceptual structure
struct SidecarManager {
    deerflow: Option<SidecarProcess>,  // Python FastAPI
    scion: Option<SidecarProcess>,     // Go service
}

struct SidecarProcess {
    child: CommandChild,       // From tauri-plugin-shell
    port: u16,                 // Allocated port
    health_status: HealthStatus,
    restart_count: u32,
    last_health_check: Instant,
}
```

The Sidecar Manager handles:
- **Startup sequencing:** Scion first (container runtime), then DeerFlow (needs Scion for sandboxes)
- **Health checks:** HTTP GET /health every 5 seconds, with exponential backoff restart on failure
- **Graceful shutdown:** On app close, signal sidecars with SIGTERM, wait 5s, then SIGKILL
- **Port allocation:** Dynamic port assignment to avoid conflicts, passed to sidecars via CLI args
- **Crash recovery:** Auto-restart with backoff (1s, 2s, 4s, 8s, max 30s), max 5 retries before surfacing error to UI

**Rationale:** Tauri's shell plugin gives Rust-side process spawning with stdout/stderr capture. The community `tauri-sidecar-manager` plugin confirms this is the standard pattern. Python/Go processes are inherently unreliable from a desktop app perspective -- they need supervision.

**Confidence:** HIGH - Tauri shell plugin docs + community patterns.

### 3. Docker Orchestration: Bollard from Rust Core

Use the **Bollard** crate (Rust Docker SDK) for all Docker operations, called from Tauri commands:

```rust
// Conceptual Docker Manager
struct DockerManager {
    client: bollard::Docker,  // Connects to local Docker socket
}

impl DockerManager {
    async fn create_agent_sandbox(&self, agent: &AgentConfig) -> Result<String> {
        // Create container with:
        // - Mounted git worktree as workspace volume
        // - Network isolation (agent-specific bridge)
        // - Resource limits (CPU, memory)
        // - Environment variables (API keys from vault, agent config)
    }

    async fn stream_container_logs(&self, container_id: &str, channel: Channel) -> Result<()> {
        // Stream container stdout/stderr through Tauri Channel to frontend
    }
}
```

**Why Bollard over shelling out to `docker` CLI:**
- Type-safe Rust API, no string parsing
- Async streaming of container logs (native integration with Tauri channels)
- No dependency on Docker CLI being in PATH
- Bollard supports Docker API 1.52, covers all needed operations

**Confidence:** HIGH - Bollard is the de facto Rust Docker SDK, 3k+ GitHub stars, actively maintained.

### 4. Git Worktrees: One Per Agent, Managed by Rust

Each agent gets an isolated git worktree so agents can modify files in parallel without conflicts:

```
project-repo/                     (main worktree)
project-repo-worktrees/
  ├── agent-architect-001/        (worktree on branch agent/architect-001)
  ├── agent-builder-002/          (worktree on branch agent/builder-002)
  ├── agent-security-003/         (worktree on branch agent/security-003)
  └── agent-qa-004/               (worktree on branch agent/qa-004)
```

**Lifecycle:**
1. **Create:** Rust Core creates worktree + branch before spawning agent container
2. **Mount:** Worktree directory is bind-mounted into the agent's Docker container
3. **Commit:** Agent commits to its branch within the container
4. **Merge:** Architect agent orchestrates merges (or user reviews conflicts in UI)
5. **Cleanup:** Rust Core removes worktree after agent completes

Use the **git2** crate for worktree operations -- no shelling out to `git` CLI.

**Shared state warning:** Worktrees share `.git/objects`. Two agents writing to the same file on different branches is safe (different worktrees). Two agents running database migrations simultaneously is NOT safe -- must be serialized.

**Confidence:** HIGH - git worktrees for parallel AI agents is a well-documented pattern (Cursor, Claude Code, Pochi all use this approach).

### 5. Flow Engine: Adapter Pattern with Trait-Based Dispatch

The `flow-engine` package is the single entry point for all 4 subsystems. It uses the **adapter pattern** with a unified interface:

```typescript
// flow-engine/src/types.ts
interface FlowEngine {
  // Intelligence (delegates to Ruflo)
  planTaskGraph(intent: ProductIntent): Promise<TaskGraph>;
  routeTask(task: Task): Promise<AgentAssignment>;

  // Execution (delegates to DeerFlow via HTTP)
  executeTask(assignment: AgentAssignment): AsyncIterable<AgentEvent>;
  cancelTask(taskId: string): Promise<void>;

  // Isolation (delegates to Scion via HTTP)
  createSandbox(config: SandboxConfig): Promise<Sandbox>;
  destroySandbox(sandboxId: string): Promise<void>;

  // Memory (delegates to EverMemOS)
  remember(context: MemoryContext): Promise<void>;
  recall(query: string): Promise<Memory[]>;
}

// Concrete adapters
class RufloAdapter implements IntelligenceProvider { ... }
class DeerFlowAdapter implements ExecutionProvider { ... }
class ScionAdapter implements IsolationProvider { ... }
class EverMemOSAdapter implements MemoryProvider { ... }

// FlowEngine composes all four
class FlowEngineImpl implements FlowEngine {
  constructor(
    private intelligence: IntelligenceProvider,
    private execution: ExecutionProvider,
    private isolation: IsolationProvider,
    private memory: MemoryProvider,
  ) {}
}
```

**Key design principle:** Ruflo and EverMemOS are direct TS imports (they run in-process). DeerFlow and Scion adapters are HTTP clients that call the sidecars. The FlowEngine hides this distinction -- consumers don't know or care which subsystem runs locally vs. as a sidecar.

**Confidence:** MEDIUM - adapter pattern is standard, but the specific subsystem APIs need validation during implementation.

### 6. Rust State Management: Managed State with Mutex

Tauri 2.0's `app.manage()` + `State<Mutex<T>>` pattern for all shared state:

```rust
struct AppState {
    sidecar_manager: SidecarManager,
    docker_manager: DockerManager,
    active_agents: HashMap<String, AgentStatus>,
    active_projects: HashMap<String, ProjectState>,
}

// In setup:
app.manage(Mutex::new(AppState::default()));

// In commands:
#[tauri::command]
async fn get_agent_status(
    state: State<'_, Mutex<AppState>>,
    agent_id: String,
) -> Result<AgentStatus, String> {
    let state = state.lock().unwrap();
    state.active_agents.get(&agent_id).cloned().ok_or("not found".into())
}
```

**Do NOT use `Arc` yourself** -- Tauri handles `Arc` wrapping internally. Use `Mutex` (or `tokio::sync::RwLock` for read-heavy state) for interior mutability.

**Confidence:** HIGH - directly from Tauri 2.0 state management docs.

### 7. Frontend State: Zustand + React Query

```
Zustand stores (client state):
  - uiStore: active studio, panel layout, theme
  - agentStore: agent statuses, live output buffers (fed by Tauri Channels)
  - projectStore: active project, file tree, git status

React Query (server state):
  - Tauri command wrappers (useQuery/useMutation around invoke())
  - Stale-while-revalidate for project data
  - Optimistic updates for user actions
```

**Why Zustand over Redux/Jotai:**
- Minimal boilerplate for a desktop app with many independent state slices
- Works naturally with Tauri's imperative IPC (no provider nesting)
- Agent output buffers need high-frequency updates without re-render storms -- Zustand's selector-based subscriptions handle this

**Why React Query for Tauri commands:**
- Commands are effectively async data fetching -- React Query's caching, deduplication, and error retry work perfectly
- `queryClient.invalidateQueries()` after mutations keeps UI consistent

**Confidence:** MEDIUM - Zustand is the community standard for Tauri + React apps, but React Query integration with Tauri invoke is a pattern, not an official recommendation.

## Component Communication Matrix

| From \ To | React UI | Rust Core | DeerFlow | Scion | Docker | Git |
|-----------|----------|-----------|----------|-------|--------|-----|
| **React UI** | - | Commands, Events | (via Rust) | (via Rust) | (via Rust) | (via Rust) |
| **Rust Core** | Channels, Events | - | HTTP | HTTP | Bollard | git2 |
| **DeerFlow** | (via Rust) | HTTP callbacks | - | HTTP | Docker SDK | git CLI |
| **Scion** | (via Rust) | HTTP callbacks | - | - | Docker SDK | git CLI |

**Critical rule:** The React frontend NEVER communicates directly with sidecars or Docker. All external communication routes through the Rust core. This ensures:
1. Security -- Rust enforces Ruflo's 7-layer Guidance Control Plane
2. Lifecycle -- Rust knows if a sidecar is down before the frontend tries to call it
3. Observability -- all IPC is traceable through one hub

## Patterns to Follow

### Pattern 1: Command + Channel Pair for Long-Running Operations

**What:** For any operation that takes >1 second, use a Command to initiate and a Channel to stream progress.

**When:** Agent execution, build processes, Docker image pulls, large file operations.

**Example:**
```typescript
// Frontend
import { invoke, Channel } from '@tauri-apps/api/core';

async function runAgent(agentConfig: AgentConfig) {
  const onEvent = new Channel<AgentEvent>();
  onEvent.onmessage = (event) => {
    // Update Zustand store with agent output
    useAgentStore.getState().appendOutput(event.agentId, event);
  };

  // Command initiates, Channel streams
  const result = await invoke('run_agent', {
    config: agentConfig,
    onEvent
  });
}
```

```rust
// Backend
#[tauri::command]
async fn run_agent(
    config: AgentConfig,
    on_event: Channel<AgentEvent>,
    state: State<'_, Mutex<AppState>>,
) -> Result<AgentResult, String> {
    // Spawn container, stream output through channel
    loop {
        let event = get_next_agent_event(&container).await?;
        on_event.send(event).map_err(|e| e.to_string())?;
        if event.is_terminal() { break; }
    }
    Ok(result)
}
```

### Pattern 2: Sidecar Health Gate

**What:** Never call a sidecar unless its health check passes. Queue or error instead.

**When:** Every DeerFlow/Scion API call.

**Example:**
```rust
impl SidecarManager {
    async fn call_deerflow<T>(&self, request: DeerFlowRequest) -> Result<T> {
        match self.deerflow_health() {
            HealthStatus::Healthy => {
                self.http_client.post(&self.deerflow_url).json(&request).send().await
            }
            HealthStatus::Starting => {
                // Wait up to 10s for health check to pass
                self.wait_for_health("deerflow", Duration::from_secs(10)).await?;
                self.http_client.post(&self.deerflow_url).json(&request).send().await
            }
            HealthStatus::Down => {
                Err(anyhow!("DeerFlow sidecar is down. Attempting restart..."))
            }
        }
    }
}
```

### Pattern 3: Multiplexed Agent Output Display

**What:** Multiple agents stream output simultaneously. Frontend demultiplexes by agent ID.

**When:** Build & Test Studio (Studio VI) with multiple agents running.

**Structure:**
```typescript
// Each agent event is tagged
interface AgentEvent {
  agentId: string;
  agentType: 'architect' | 'builder' | 'security' | 'qa';
  eventType: 'stdout' | 'stderr' | 'status' | 'commit' | 'error';
  payload: string;
  timestamp: number;
}

// Zustand store with per-agent buffers
interface AgentStore {
  agents: Map<string, {
    status: AgentStatus;
    outputBuffer: AgentEvent[];  // Ring buffer, max 10k events
    lastActivity: number;
  }>;
  appendOutput: (agentId: string, event: AgentEvent) => void;
}
```

## Anti-Patterns to Avoid

### Anti-Pattern 1: Frontend-to-Sidecar Direct Communication

**What:** React UI making HTTP calls directly to DeerFlow/Scion localhost ports.

**Why bad:** Bypasses Rust security layer, no lifecycle awareness (sidecar might be restarting), no centralized error handling, leaks implementation detail (port numbers) to frontend.

**Instead:** All sidecar calls go through Tauri commands. The frontend knows "agents" and "sandboxes", not "DeerFlow" and "Scion."

### Anti-Pattern 2: Events for Streaming Agent Output

**What:** Using `emit()` / `listen()` for high-frequency agent output.

**Why bad:** Tauri's event system is explicitly not designed for high throughput. It will drop messages under load and has no ordering guarantee.

**Instead:** Use Tauri Channels (`Channel<T>`), which are designed for streaming, ordered delivery, and support raw byte payloads.

### Anti-Pattern 3: Single Git Branch for All Agents

**What:** All agents commit to the same branch concurrently.

**Why bad:** Merge conflicts on every commit, agents block each other, impossible to roll back one agent's work.

**Instead:** One worktree + branch per agent. Architect agent orchestrates merges.

### Anti-Pattern 4: Monolithic Rust Core

**What:** All Tauri command logic in a single `lib.rs` or `main.rs`.

**Why bad:** 500+ line files, impossible to test, tight coupling between Docker/Git/Sidecar concerns.

**Instead:** Module-per-concern in the Rust `src-tauri/src/` directory:
```
src-tauri/src/
  ├── main.rs              (app setup, plugin registration)
  ├── state.rs             (AppState definition)
  ├── commands/
  │   ├── mod.rs
  │   ├── agents.rs        (agent lifecycle commands)
  │   ├── projects.rs      (project CRUD commands)
  │   ├── docker.rs        (container management commands)
  │   └── git.rs           (worktree/branch commands)
  ├── sidecar/
  │   ├── mod.rs
  │   ├── manager.rs       (lifecycle, health checks)
  │   ├── deerflow.rs      (DeerFlow HTTP client)
  │   └── scion.rs         (Scion HTTP client)
  ├── docker/
  │   ├── mod.rs
  │   └── manager.rs       (Bollard wrapper)
  └── git/
      ├── mod.rs
      └── worktree.rs      (git2 worktree operations)
```

### Anti-Pattern 5: Bundling Python/Go Source into Tauri

**What:** Trying to compile DeerFlow (Python) or Scion (Go) into the Tauri binary.

**Why bad:** Tauri bundles Rust + WebView. Python requires a runtime. Go compiles to a separate binary. Neither can be linked into the Tauri process.

**Instead:** DeerFlow is packaged with PyInstaller into a standalone binary. Scion is `go build` into a standalone binary. Both are declared as `externalBin` in `tauri.conf.json` and shipped as sidecars.

## Scalability Considerations

| Concern | 1-3 Agents | 5-8 Agents | 12 Agents (full swarm) |
|---------|------------|------------|------------------------|
| **Memory** | ~2GB (Tauri + 2 sidecars + containers) | ~6GB (more containers) | ~12GB+ (need resource limits per container) |
| **Disk (worktrees)** | ~500MB per project | ~2GB per project | ~5GB per project (worktrees share .git/objects) |
| **IPC throughput** | Single Channel sufficient | Multiplexed Channel with agent IDs | May need per-agent Channels if throughput exceeds single Channel capacity |
| **Docker** | Default Docker Desktop sufficient | Monitor Docker Desktop memory allocation | Need Docker resource governance (CPU/mem limits per container) |
| **Git operations** | git2 handles fine | Concurrent worktree ops need serialization for safety | Worktree creation/deletion should be queued, not parallel |

## Build Order (Dependency Chain)

The architecture has clear dependency ordering that informs the roadmap:

```
Phase 1: Foundation (no external dependencies)
  ├── Tauri shell + React scaffold
  ├── Rust module structure (commands/, sidecar/, docker/, git/)
  ├── Tauri IPC layer (commands + channels + events)
  ├── SQLite persistence (tauri-plugin-sql)
  └── Frontend state (Zustand + React Query + Tauri invoke wrappers)

Phase 2: Process Management (depends on Phase 1)
  ├── Sidecar Manager (spawn, health check, restart, shutdown)
  ├── DeerFlow sidecar packaging (PyInstaller) + HTTP client
  ├── Scion sidecar packaging (go build) + HTTP client
  └── Sidecar health gating in commands

Phase 3: Execution Environment (depends on Phase 2)
  ├── Docker Manager via Bollard
  ├── Git worktree manager via git2
  ├── Container <-> worktree binding
  └── Agent output streaming (Container -> Scion -> Rust -> Channel -> React)

Phase 4: Intelligence Layer (depends on Phase 2, parallel with Phase 3)
  ├── Ruflo adapter (direct TS import)
  ├── EverMemOS adapter (direct TS import)
  ├── Flow Engine unified interface
  └── Task graph planning + agent routing

Phase 5: Studios (depends on Phases 3 + 4)
  ├── Studio I: Product Requirements (PRD wizard)
  ├── Studio III: Planning (GitHub connection, build plan)
  └── Studio VI: Build & Test (agent live feed, file explorer, terminal)
```

**Critical path:** Phase 1 -> Phase 2 -> Phase 3 -> Phase 5. The intelligence layer (Phase 4) can be built in parallel with the execution environment because Ruflo and EverMemOS are TS packages that don't depend on Docker/git infrastructure.

## Sources

- [Tauri 2.0 IPC Concepts](https://v2.tauri.app/concept/inter-process-communication/) - Commands, Events, Channels
- [Tauri 2.0 Calling Frontend from Rust](https://v2.tauri.app/develop/calling-frontend/) - Channel streaming
- [Tauri 2.0 State Management](https://v2.tauri.app/develop/state-management/) - Mutex + manage() pattern
- [Tauri Shell Plugin](https://v2.tauri.app/plugin/shell/) - Sidecar spawning and lifecycle
- [Tauri Sidecar Embedding](https://v2.tauri.app/develop/sidecar/) - externalBin configuration
- [Bollard - Docker SDK for Rust](https://github.com/fussybeaver/bollard) - Container management from Rust
- [tauri-sidecar-manager](https://github.com/radical-data/tauri-sidecar-manager) - Community sidecar lifecycle plugin
- [Git Worktrees for Parallel AI Agents](https://dev.to/getpochi/how-we-built-true-parallel-agents-with-git-worktrees-2580) - Worktree isolation pattern
- [Tauri + Python Sidecar Example](https://github.com/dieharders/example-tauri-v2-python-server-sidecar) - Reference implementation
- [Tauri Sidecar Lifecycle Feature Request](https://github.com/tauri-apps/plugins-workspace/issues/3062) - Community patterns for health checks
