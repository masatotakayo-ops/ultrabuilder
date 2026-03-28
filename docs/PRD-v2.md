# Ultrabuilder — Product Requirements Document v2

**Version:** 2.0
**Date:** 2026-03-28
**Status:** Approved — Implementation Ready
**Supersedes:** ultrabuilder-prd.md (v1, original vision doc)

---

## What Has Changed Since v1

v1 was a vision document. v2 reflects all architectural decisions, stack selections, phase planning, and research-derived enhancements made during the specification phase. Every section has been updated or confirmed against implementation plans.

**Material changes from v1:**
- Tech stack changed: Next.js web → **Tauri 2.0 desktop app** (rationale in §4)
- 5 OSS foundations confirmed with exact integration strategy
- 56 v1 requirements formalized (up from unstructured feature list)
- 6-phase delivery roadmap with parallel execution paths
- New: Agent capability manifests (INTL-07)
- New: Living CONVENTIONS.md pattern (BILD-09)
- New: File-context routing for Builder agent (INTL-02 enhanced)
- New: EverMemOS HNSW quantized vector storage for memory efficiency (INTL-04 enhanced)
- Phase 1 UI/UX decisions locked (VS Code shell, Drizzle proxy, IPC layer design)

---

## 1. Product Overview

### 1.1 What It Is

Ultrabuilder is an **autonomous software factory** — a Tauri desktop application that turns product intent into production-ready commercial SaaS through wizard-driven Connected Studios, a coordinated agent swarm, and deep GitHub integration.

Users define what they want to build. The platform's Flow Engine (Ruflo + DeerFlow + Scion + EverMemOS) plans, generates, tests, and deploys the software into a GitHub repository. Built for developers and teams who want AI-powered software synthesis with full traceability.

### 1.2 Core Value

> User describes what they want → the system plans it → agents build it into a GitHub repo with real, working code.

This is the golden path. Every architectural and product decision traces back to making this pipeline work. Everything else is secondary.

### 1.3 What It Is Not

Ultrabuilder is not:
- A browser-based IDE (Bolt/Lovable own that lane)
- A code autocomplete tool (Cursor/Copilot own that lane)
- A generic chat assistant (commoditized)
- A "vibe coding" tool (structured pipeline is the explicit anti-thesis of prompt-and-pray)
- A hosting provider (push to GitHub, users deploy via their own CI/CD)

### 1.4 Competitive Position

| Competitor | Their Lane | Ultrabuilder's Advantage |
|------------|-----------|--------------------------|
| Cursor / Windsurf | Best-in-class editing UX | Factory, not editor. Structured pipeline vs freestyle. |
| Devin | Full autonomy, PR lifecycle | Adds PRD/planning stages Devin skips. 12 specialized agents vs 1 general-purpose. |
| Bolt / Lovable | Zero-setup, prompt-to-app | Produces production software. Desktop power, local execution, Docker isolation. |
| GitHub Copilot | Ecosystem reach, issue-to-PR | Copilot is one station; Ultrabuilder is the full pipeline. |
| v0.dev | React/Next.js UI generation | Stack-agnostic, full lifecycle, not just UI. |

**Unique differentiators no competitor has:**
1. Structured PRD-first pipeline (Studios I → III → VI)
2. 12 specialized role agents (Architect, Discoverer, Builder, Security, QA, etc.)
3. Persistent cross-session memory at 93% LoCoMo accuracy (EverMemOS)
4. Security-by-default via 7-layer Guidance Control Plane (Ruflo)
5. Agent capability manifests — explicit per-agent tool access and prohibited actions
6. Living CONVENTIONS.md — Architect captures and Builder consumes project conventions
7. Local-first, offline-capable desktop (no cloud dependency for core pipeline)

---

## 2. Target Users

### Primary: Senior developers and technical teams

- Building commercial SaaS products
- Want AI assistance that respects their codebase conventions and architecture decisions
- Need auditability — every agent action must be traceable
- Work on macOS, Windows, or Linux

### Secondary: Technical founders / solo developers

- Building production-grade software without a full team
- Need a system that handles architecture and quality, not just code generation
- Value the structured pipeline that prevents technical debt by design

### Not targeting (M1):
- Non-technical users (no wizard simplification layer)
- Enterprise teams (SSO/SAML/SCIM deferred to Phase 03)
- Mobile developers (no mobile app, no mobile-specific agents)

---

## 3. Open-Source Foundations

Ultrabuilder is built on 5 forked open-source systems, integrated as a unified Flow Engine:

| System | Role | Repo | Language | License | Integration |
|--------|------|------|----------|---------|-------------|
| **Ruflo** | Intelligence — routing, policy, learning | github.com/ruvnet/ruflo | TypeScript | Apache 2.0 | Direct TS adapter in `packages/ruflo` |
| **DeerFlow** | Execution — Docker sandboxes, sub-agents | github.com/bytedance/deer-flow | Python | MIT | Python sidecar + TS SDK client |
| **Scion** | Isolation — containers, worktrees, lifecycle | github.com/GoogleCloudPlatform/scion | Go | Apache 2.0 | Go sidecar + TS SDK client |
| **EverMemOS** | Memory — MemCells, HNSW recall, consolidation | github.com/EverMind-AI/EverMemOS | Python | Apache 2.0 | Direct TS adapter in `packages/evermemos` |
| **MiroFish** | Prediction — swarm simulation | github.com/666ghj/MiroFish | Python | **AGPL-3.0** | Phase 02 — requires process isolation + legal review |

**AGPL note:** MiroFish's AGPL-3.0 license requires any modification to be open-sourced. It must run in an isolated process with no code sharing with the commercial codebase. Legal counsel required before Phase 02 integration.

---

## 4. Technology Stack

### 4.1 Platform Decision: Tauri 2.0 Desktop

The original v1 PRD specified a Next.js web app. **This was changed to Tauri 2.0 desktop** during architecture review. Rationale:

| Requirement | Next.js Web | Tauri Desktop |
|-------------|-------------|---------------|
| File system access | ❌ Browser sandbox | ✓ Native OS access |
| Docker orchestration | ❌ Not possible | ✓ bollard Rust crate |
| Git worktrees | ❌ Not possible | ✓ git2 Rust crate |
| Long-running agents | ❌ 10s serverless limit | ✓ No timeout |
| Local-first / offline | ❌ Requires server | ✓ Works offline |
| IDE-like canvas | ⚠ Possible but complex | ✓ Native WebView |

Every successful AI coding tool (Cursor, Windsurf, VS Code) is desktop. The IDE-like requirement is non-negotiable for Ultrabuilder's feature set.

### 4.2 Full Stack

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| **Desktop shell** | Tauri | 2.10.x | Rust backend + WebView frontend |
| **Frontend language** | TypeScript | 6.0.x | Strict mode, all packages |
| **Backend language** | Rust (stable) | latest | Tauri commands, Docker, git |
| **UI framework** | React | 19.2.x | Component model, streaming |
| **Styling** | Tailwind CSS | 4.x | CSS-first config (@theme) |
| **Component library** | shadcn/ui | CLI v4 | Components copied into project |
| **State management** | Zustand | 5.x | Slices pattern for IDE state |
| **Async state** | TanStack Query | 5.x | Caching, background refetch |
| **Routing** | TanStack Router | 5.x | File-based, type-safe, hash mode |
| **Code editor** | Monaco (React wrapper) | 4.7.x | VS Code editor, diff view |
| **Architecture canvas** | React Flow (@xyflow/react) | 12.10.x | Agent workflow visualization |
| **Terminal emulator** | xterm.js | 6.x | Agent output streams, build logs |
| **Local API sidecar** | Hono | 4.12.x | Compiled to single binary (sidecar) |
| **ORM** | Drizzle | 0.45.x | Proxy pattern — frontend computes SQL, Rust executes |
| **Local database** | SQLite (tauri-plugin-sql) | 2.3.x | Local-first, Rust-side via sqlx |
| **Cloud database** | PostgreSQL 16+ | — | Team sync (Phase 02+) |
| **Auth** | Clerk | latest | Email/password only in Tauri (OAuth broken in WebView) |
| **Monorepo** | Turborepo | 2.8.x | Incremental builds, shared cache |
| **Bundler** | Vite | 6.x | Frontend bundler + Tauri default |
| **Linter/formatter** | Biome | 2.x | Single binary, 25x faster than ESLint+Prettier |
| **Testing** | Vitest | 4.x | Unit/integration, Jest-compatible |
| **E2E testing** | Playwright | latest | Cross-platform desktop E2E |
| **Docker API (Rust)** | bollard | 0.x | Type-safe Docker API from Tauri commands |
| **Git API (Rust)** | git2 | 0.19.x | libgit2 bindings — worktree management |
| **Sidecar packaging** | @yao-pkg/pkg | latest | Compile Hono sidecar to single binary |

### 4.3 Monorepo Structure

```
ultrabuilder/
├── apps/
│   └── desktop/              ← Tauri 2.0 shell + React UI
│       ├── src/              ← React frontend (TypeScript)
│       └── src-tauri/        ← Rust backend (commands, plugins)
├── packages/
│   ├── ruflo/                ← Intelligence layer (forked TS adapter)
│   ├── deerflow/             ← Execution harness (API client SDK)
│   ├── scion/                ← Container runtime (API client SDK)
│   ├── evermemos/            ← Memory OS (forked TS adapter)
│   ├── flow-engine/          ← Unified adapter over all 4 systems
│   └── shared/               ← Types, schema.ts, constants, utilities
├── turbo.json
└── package.json
```

**Package manager:** pnpm (strict hoisting, workspace protocol, 2-3x faster installs)

### 4.4 IPC Architecture (Three-Tier)

| Tier | Mechanism | Use Case |
|------|-----------|----------|
| **Commands** | `#[tauri::command]` → `invoke()` | Request/response — DB CRUD, system calls |
| **Channels** | `tauri::ipc::Channel` → `useChannel()` | Streaming — agent output, log streams, real-time data |
| **Events** | `app.emit()` → `listen()` | Lifecycle — app:ready, sidecar:started, sidecar:crashed |

### 4.5 Drizzle Proxy Pattern

Drizzle ORM cannot use native SQLite drivers inside Tauri's WebView. The proxy pattern resolves this:

1. Frontend writes type-safe queries using Drizzle's query builder
2. Drizzle proxy driver serializes the query to `{ sql, params }`
3. Frontend calls `invoke("run_sql", { sql, params })`
4. Rust executes via `tauri-plugin-sql` / sqlx, returns rows
5. Drizzle deserializes the result

**Schema location:** `packages/shared/src/schema.ts` — one type definition shared across DB → API → UI.

---

## 5. App Shell (Phase 1 Decisions)

The following UI/UX decisions are locked for Phase 1 and establish the permanent skeleton all future phases build on.

### 5.1 Layout

**VS Code-style layout:**
```
┌─────────────────────────────────────────────────────┐
│ Custom Titlebar [drag region]   [−][□][×]           │
├──┬──────────────┬────────────────────────────────────┤
│  │              │                                    │
│A │  Sidebar     │  Main Editor Area                  │
│c │  (collaps-   │  (Phase 1: Projects screen)        │
│t │  ible)       │  (Phase 5: Studio content)         │
│. │              │                                    │
│B │              │                                    │
│a │              │                                    │
│r │              ├────────────────────────────────────┤
│  │              │  Bottom Panel (Log | IPC Debug)    │
└──┴──────────────┴────────────────────────────────────┘
```

- **Activity bar** (far left): Icon-based navigation to sections
- **Sidebar**: Collapsible from day one via toggle; expands/collapses without page reload
- **Main editor area**: Phase 1 shows Projects screen; Phase 5 slots Studio UIs here
- **Bottom panel**: Log streaming + dev-only IPC debug panel

### 5.2 Phase 1 Content: Projects Screen

The Phase 1 main area is a **Projects screen** — list of local projects with name, description, status, and date. Actions: Create Project, Open Project, Delete Project. No splash screen — land directly on projects list on launch.

### 5.3 Routing

TanStack Router (hash mode to avoid Tauri custom protocol issues) configured from Phase 1. Example route structure:
```
/ → Projects screen
/projects/:id → Project workspace
/projects/:id/studio/prd → PRD Studio (Phase 5)
/projects/:id/studio/plan → Planning Studio (Phase 5)
/projects/:id/studio/build → Build Studio (Phase 5)
```

### 5.4 Window Chrome

| Property | Decision |
|----------|----------|
| **Titlebar** | Custom (Tauri `decorations: false`), draggable via `data-tauri-drag-region` CSS |
| **Native menu** | Full IDE menu — File (New/Open/Quit), Edit (Undo/Redo/Cut/Copy/Paste), View (Toggle Sidebar/Bottom Panel/Zoom), Window (Minimize/Zoom/Close), Help (About) |
| **Theme** | System-aware — respects `prefers-color-scheme`, both light and dark fully implemented |
| **Typography** | Geist Sans (UI text) + Geist Mono (code, paths, IDs, timestamps, IPC debug) — bundled, no CDN |

### 5.5 IPC Proof Points

Phase 1 proves all three IPC primitives with real, production-useful implementations:

| Primitive | Phase 1 Implementation | Reused In |
|-----------|----------------------|-----------|
| **Commands** | DB CRUD: `create_project`, `list_projects`, `update_project`, `delete_project` | Phase 5 (all Studio operations) |
| **Channels** | Log streaming: Rust tracing events → bottom-panel console | Phase 3 (agent output streaming) |
| **Events** | App lifecycle: `app:ready`, `window:focus`, `window:blur` | Phase 2 extends with `sidecar:started`, `sidecar:crashed` |

**IPC debug panel:** Visible in `NODE_ENV=development` only. Shows live IPC events (type, direction, payload, timestamp) — hidden in production builds.

### 5.6 Initial SQLite Schema

Phase 1 creates one table: `projects`. All other entities (agent_runs, secrets, build_plans) are added via migrations in their respective phases.

```typescript
// packages/shared/src/schema.ts
projects: {
  id: uuid (PK)
  name: text (not null)
  description: text (nullable)
  github_repo: text (nullable)       // wired in Phase 5
  status: enum(planning|building|complete)
  created_at: timestamp
  updated_at: timestamp
}
```

Drizzle migrations run on app startup. Migration files committed to repo.

---

## 6. Intelligence Layer Design

### 6.1 Flow Engine

A unified TypeScript interface (`packages/flow-engine`) that abstracts all four active OSS systems:

```typescript
interface FlowEngine {
  route(task: AgentTask): Promise<AgentResult>      // Ruflo routing
  execute(plan: ExecutionPlan): AsyncIterable<Event> // DeerFlow execution
  isolate(config: SandboxConfig): Promise<Container> // Scion isolation
  remember(cell: MemCell): Promise<void>             // EverMemOS store
  recall(query: MemQuery): Promise<MemCell[]>        // EverMemOS search
}
```

### 6.2 Ruflo Routing — File-Context Enhancement

Ruflo's neural routing passes file-type context to agents. The Builder agent receives language-specific instruction profiles based on what it's generating:

| File Type | Builder Instruction Profile |
|-----------|----------------------------|
| TypeScript (.ts/.tsx) | TS strict mode conventions, shadcn patterns, React 19 patterns |
| Python (.py) | PEP 8, type hints, async patterns |
| Rust (.rs) | Error handling patterns, ownership conventions |
| YAML/TOML | Schema validation patterns, project conventions |

This is the application of the GitHub Copilot `applyTo` pattern at the routing layer — instructions are scoped to file type, not applied globally.

### 6.3 Agent Capability Manifests (INTL-07)

Every agent has an explicit capability manifest enforced by the Guidance Control Plane:

| Agent | Allowed | Prohibited |
|-------|---------|-----------|
| **Architect** | Read all files, write architecture docs, create CONVENTIONS.md | Direct code write, test execution |
| **Builder** | Write code, run tests, create branches | Merge to main, modify architecture docs |
| **Security** | Read all files, run linters/scanners, write security reports | Write source code |
| **QA** | Write tests, run test suite, write QA reports | Modify source code (only test files) |
| **Discoverer** | Read all, search external ecosystems, write research reports | Write code |

These are enforced at the Ruflo Guidance Control Plane level — agents cannot bypass their manifest through prompt injection or tool misuse.

### 6.4 CONVENTIONS.md Pattern (BILD-09)

The Architect agent creates and maintains `CONVENTIONS.md` in the target repo. This solves the core "context problem" identified in GitHub's agent customization research: agents produce off-target output because they don't know team conventions.

**Lifecycle:**
1. Architect agent creates initial `CONVENTIONS.md` after architecture design
2. File captures: naming patterns, framework choices, testing conventions, import styles, file organization
3. Architect updates it at each checkpoint when new patterns emerge
4. Builder agent **reads `CONVENTIONS.md` before every code generation task** — it is part of `read_first` in every Builder task plan
5. Git history provides full traceability of when conventions were established and changed

**Example content:**
```markdown
# Project Conventions

## Naming
- Components: PascalCase (UserProfile.tsx)
- Hooks: camelCase with `use` prefix (useProjectStore.ts)
- API handlers: camelCase with HTTP method prefix (getProjects.ts)

## Testing
- Unit tests: co-located with source (UserProfile.test.tsx)
- Integration tests: tests/integration/
- All tests must pass before merge

## Framework choices
- State: Zustand slices (not Context API)
- API calls: TanStack Query (not fetch directly)
```

### 6.5 EverMemOS Memory — HNSW Quantization

EverMemOS uses HNSW (Hierarchical Navigable Small World) for vector similarity search across MemCells. Based on TurboQuant research (ICLR 2026), the HNSW index must be configured with **quantized vector storage (PQ or SQ compression)**:

- **PQ (Product Quantization):** Splits high-dimensional vectors into sub-vectors, quantizes each independently. ~8-16x memory reduction with ~95% recall preservation.
- **SQ (Scalar Quantization):** Quantizes each dimension independently. Simpler, ~4-8x reduction with ~98% recall.

**Requirement:** Phase 4 integration of EverMemOS must validate that chosen quantization configuration preserves ≥ 90% recall accuracy compared to unquantized baseline.

---

## 7. Connected Studios

### 7.1 Studio I — Product Requirements (PRD Wizard)

The entry point to the entire pipeline. Without structured intent, everything downstream is garbage-in/garbage-out.

**User flow:**
1. User launches PRD Wizard from Projects screen
2. Wizard guides through: product description → target users → core features → constraints → success criteria
3. Output: structured `prd.md` file saved to project
4. User can edit and re-run at any time

**Requirements:** PRD-01, PRD-02, PRD-03, PRD-04

### 7.2 Studio III — Planning

Bridges product intent to agent execution.

**User flow:**
1. User connects GitHub repository (PAT or GitHub App)
2. System verifies read/write access
3. Build plan generated from PRD — features broken into ordered agent tasks
4. User reviews and approves plan
5. Secrets vault initialized — API keys/tokens stored encrypted, injected automatically at runtime

**Requirements:** PLAN-01, PLAN-02, PLAN-03, PLAN-04, PLAN-05, PLAN-06

### 7.3 Studio VI — Build & Test

Where users see value. The IDE-like environment where agents build the software.

**UI components:**
- **File explorer** (left sidebar): Real-time project file tree as agents generate code
- **Agent live feed** (right sidebar): Streaming output from all active agents, per-agent filtering
- **Monaco editor** (main area): Generated code with syntax highlighting and diff view
- **Integrated terminal** (bottom panel): Build output, manual command execution
- **Test runner** (bottom panel tab): Inline test results

**Agent behaviors:**
- Every action commits to GitHub with structured message: `[agent-name] action: description`
- User can pause, retry, or override any agent at checkpoints
- Error detection loop: agents detect test/build failures and auto-fix without user intervention
- Architect maintains `CONVENTIONS.md` at each checkpoint

**Requirements:** BILD-01 through BILD-09

---

## 8. Agent Swarm

### 8.1 Core Agents (M1 — 4 active)

| Agent | Capability Manifest | Primary Responsibility |
|-------|-------------------|----------------------|
| **Architect** | Read-all + write architecture docs | Creates target architecture from PRD, maintains CONVENTIONS.md |
| **Builder** | Write code + run tests + read CONVENTIONS.md | Generates code per architecture and policy gates |
| **Security** | Read-all + scanners, no code write | Continuously scans generated code for vulnerabilities |
| **QA** | Write tests + run suite, no source modification | Generates and executes tests, produces build confidence score |

### 8.2 Extended Swarm (Phase 5)

Additional agents from the 12-agent swarm may be activated as needed:
- **Discoverer** — source discovery and ecosystem search (v2)
- **DevOps** — deployment pipeline configuration
- **Documenter** — API docs, README generation

### 8.3 Coordination Protocol

- Architect agent is authoritative — explicit state machine governs transitions
- Agents run in parallel where task dependencies allow
- Single-writer rule: no two agents can write the same file simultaneously
- Every agent decision logged with full context in git history
- Agent traceability: commit messages include agent identity and decision rationale

---

## 9. GitHub Integration

GitHub is the single source of truth for all generated software. All agent work flows through git.

| Requirement | Behavior |
|------------|---------|
| **Connection** | Personal access token or GitHub App |
| **Commit format** | `[agent-name] action: description` for every agent action |
| **Branching** | Each agent gets a dedicated git worktree on a feature branch |
| **Merging** | Completed features merged via automated PR creation |
| **Governance** | Branch protection rules and existing policies respected — agents work within, not around |
| **Worktree cleanup** | Automatic on task completion |

---

## 10. Security

### 10.1 Guidance Control Plane (Ruflo)

7-layer policy gates enforced at the orchestration level. Agents cannot bypass through prompt injection or tool misuse. Gates run before any agent action is executed.

### 10.2 Agent Capability Manifests

Per-agent tool access lists and prohibited action lists (see §6.3). Enforced by the Control Plane.

### 10.3 Secrets Management

- Encrypted vault scoped at workspace/project/environment levels
- Secrets injected into agent execution environments automatically — no mid-build prompts
- Never stored in plaintext, never committed to git
- Environment-specific scoping (dev/staging/prod)

### 10.4 Container Isolation

All agent code execution happens inside isolated Docker containers via Scion:
- Resource limits enforced (CPU, memory) via bollard
- No two agents share a container
- Network access controlled per-container
- Containers destroyed on task completion

---

## 11. Authentication

**M1:** Single-user, email/password via Clerk.

**Architecture:** Organizations model initialized from day one (team-ready, not yet exposed in UI).

**Tauri constraint:** OAuth flows are broken in WebView. Email/password is the only supported auth method in Tauri until a patched flow is implemented.

**Session persistence:** User session persists across app restarts.

---

## 12. Delivery Roadmap

### Phase Overview

| Phase | Goal | Requirements | Parallel? |
|-------|------|--------------|-----------|
| **1** | Tauri desktop app, React shell, SQLite, IPC, CI | FNDN-01–05 | No (first) |
| **2** | DeerFlow + Scion as supervised sidecar processes | SIDE-01–06 | No |
| **3** | Docker + git worktrees, agent streaming | EXEC-01–06 | ✓ with Phase 4 |
| **4** | Flow Engine (Ruflo + EverMemOS), agent protocols | INTL-01–07 | ✓ with Phase 3 |
| **5** | Connected Studios (PRD, Planning, Build & Test) | PRD, PLAN, BILD, AGNT, GIT | No |
| **6** | Clerk auth, Organizations, release distribution | AUTH-01–03 | No |

### Phase 1 (Current): Foundation and Shell

**5 execution plans, 4 waves:**

| Wave | Plans | Builds |
|------|-------|--------|
| 1 | 01-01 | Turborepo monorepo scaffold, pnpm workspaces, packages/shared, Biome |
| 2 | 01-02 | Tauri app + VS Code shell, custom titlebar, IDE menu, TanStack Router, Geist, system theme |
| 3 | 01-03, 01-04 (parallel) | SQLite + Drizzle CRUD **and** Channels + Events + IPC debug panel |
| 4 | 01-05 | GitHub Actions CI matrix (macOS ARM, Intel, Ubuntu, Windows) + Vitest |

**Phase 1 success criteria:**
1. User can launch the desktop app on macOS, Windows, and Linux and see the React UI
2. App persists project data locally across restarts via SQLite
3. Monorepo builds all packages successfully with a single command
4. CI pipeline runs and passes on all three OS targets
5. Frontend can invoke Rust commands, receive streaming data via Channels, and react to lifecycle Events

---

## 13. Requirements Summary (v1 — 56 total)

### Foundation (Phase 1) — 5 requirements
FNDN-01 through FNDN-05: Tauri app, Turborepo monorepo, SQLite + Drizzle, cross-platform CI, IPC layer

### Sidecar Infrastructure (Phase 2) — 6 requirements
SIDE-01 through SIDE-06: DeerFlow/Scion sidecars, supervisor with backoff, bundled executables, Docker onboarding

### Execution Environment (Phase 3) — 6 requirements
EXEC-01 through EXEC-06: Docker via bollard, isolated containers with resource limits, git worktrees via git2, agent output streaming, cleanup, single-writer rule

### Intelligence Layer (Phase 4) — 7 requirements
- INTL-01: Unified Flow Engine TypeScript interface
- INTL-02: Ruflo neural routing + file-type context → language-specific Builder profiles
- INTL-03: 7-layer Guidance Control Plane
- INTL-04: EverMemOS MemCells + HNSW with PQ/SQ quantization (≥90% recall)
- INTL-05: Hierarchical agent coordination (Architect authoritative, explicit state machine)
- INTL-06: OpenAI / Anthropic / Google model flexibility
- INTL-07: Per-agent capability manifests (Architect/Builder/Security/QA boundaries)

### Studio I (Phase 5) — 4 requirements
PRD-01 through PRD-04: PRD wizard, structured output, content spec, re-runnable

### Studio III (Phase 5) — 6 requirements
PLAN-01 through PLAN-06: GitHub connection, access verification, build plan, approval, secrets vault, secret injection

### Studio VI (Phase 5) — 9 requirements
- BILD-01–08: File explorer, agent live feed, GitHub commits, pause/retry/override, terminal, test runner, error loop, Monaco editor
- BILD-09: Architect maintains CONVENTIONS.md; Builder reads it before every code generation task

### Agent Swarm (Phase 5) — 6 requirements
AGNT-01 through AGNT-06: Architect/Builder/Security/QA agents, parallel execution, full traceability

### GitHub Integration (Phase 5) — 4 requirements
GIT-01 through GIT-04: Connection, structured commits, automated PRs, governance respect

### Authentication (Phase 6) — 3 requirements
AUTH-01 through AUTH-03: Email/password via Clerk, session persistence, Organizations model

---

## 14. Deferred to v2 (Phase 02+)

| Feature | Why Deferred |
|---------|-------------|
| Source Discovery & Qualification | High value, high complexity. Core pipeline works without it. |
| Architecture Synthesis engine | Depends on Source Discovery. |
| MiroFish / Commercialization Studio | AGPL-3.0 legal review required. Not blocking core value. |
| Architecture Studio (canvas) | Nice-to-have visualization layer. |
| Interface Design Studio | Depends on Theme Engine. |
| Production Deployment Studio | Manual deploy (push to GitHub) sufficient for M1. |
| Team collaboration (roles, concurrency, review flows) | Single-user first; architecture is team-ready. |
| Slack / Linear integrations | Not blocking core value. |
| Multi-channel agent input (Telegram, WhatsApp, Feishu) | Phase 02+. |
| Plugin marketplace | Zero users, zero plugins — premature. |
| Enterprise features (SSO/SAML/SCIM) | Phase 03. |

---

## 15. Key Architectural Decisions Log

| Decision | Rationale | Date |
|----------|-----------|------|
| Tauri desktop over Next.js web | IDE-like requirements demand OS-level access: file system, Docker, git, long-running processes. Every successful AI coding tool is desktop. | 2026-03-28 |
| Turborepo monorepo | Single repo for desktop app + all forked packages. Shared types, incremental builds, one push deploys. | 2026-03-28 |
| pnpm package manager | Strict hoisting, workspace protocol, 2-3x faster installs, native Turborepo support. | 2026-03-28 |
| Drizzle proxy pattern | Drizzle ORM in Tauri WebView: frontend computes SQL, Rust executes via sqlx. Only viable pattern for type-safe DB access in Tauri. | 2026-03-28 |
| DeerFlow/Scion as sidecars | Python/Go cannot be bundled in Tauri WebView. Run as local API services, TypeScript SDK wraps calls. | 2026-03-28 |
| Vertical slice milestone (Studios I→III→VI) | Core pipeline end-to-end before peripheral studios. Proves core value fastest. | 2026-03-28 |
| Clerk for auth (email/password only) | Organizations model matches PRD role system. OAuth broken in Tauri WebView — email/password only for M1. | 2026-03-28 |
| Local-first + cloud sync architecture | Desktop works offline. Agents run locally, zero latency. Team sync via API when connected. | 2026-03-28 |
| VS Code-style shell layout | Activity bar → collapsible sidebar → editor area → bottom panel. Permanent skeleton all future phases slot into. | 2026-03-28 |
| Agent capability manifests | GitHub research shows "context problem, not capability problem." Explicit per-agent boundaries enforced by Guidance Control Plane prevent tool misuse and improve output quality. | 2026-03-28 |
| CONVENTIONS.md pattern | Architect captures project conventions; Builder reads them before code generation. Solves the root cause of off-target output: agents don't know team standards. | 2026-03-28 |
| EverMemOS HNSW quantization | TurboQuant (ICLR 2026) demonstrates PQ/SQ vector quantization achieves 8-16x memory reduction with ≥95% recall. Applied to EverMemOS HNSW index to reduce MemCell memory footprint and speed similarity search. | 2026-03-28 |
| MiroFish deferred (AGPL) | AGPL-3.0 requires process isolation and legal review before any commercial integration. No Phase 01 risk. | 2026-03-28 |

---

*PRD v2 — 2026-03-28*
*Supersedes ultrabuilder-prd.md*
*Next update: after Phase 1 execution (FNDN requirements validated)*
