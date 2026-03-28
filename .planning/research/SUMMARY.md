# Project Research Summary

**Project:** Ultrabuilder
**Domain:** Autonomous Software Factory / AI-Powered IDE Desktop App
**Researched:** 2026-03-28
**Confidence:** HIGH

## Executive Summary

Ultrabuilder is a desktop-native AI-powered software factory built on Tauri 2, orchestrating 12 specialized agents through a structured pipeline (PRD to Planning to Build to Deploy). The competitive landscape is split between browser-based generators (Bolt, Lovable, Replit) and AI-augmented editors (Cursor, Windsurf, Copilot). Ultrabuilder occupies a distinct lane: it is neither a typing assistant nor a prompt-and-pray generator. Its differentiator is the structured pipeline from product intent to production code, powered by multi-agent coordination with persistent memory. Experts building this class of product use Tauri 2 for the desktop shell (avoiding Electron's memory overhead), Monaco for the code editor, Docker containers for agent sandboxing, and git worktrees for parallel agent isolation. The recommended stack is mature and high-confidence across the board.

The recommended approach is a hub-and-spoke architecture with the Tauri Rust core as the single coordination point for all process lifecycles, IPC routing, and security enforcement. DeerFlow (Python) and Scion (Go) run as supervised sidecar processes, not as embedded libraries. The Flow Engine provides a unified TypeScript adapter over all four subsystems (Ruflo, DeerFlow, Scion, EverMemOS), hiding the distinction between in-process TS packages and HTTP-based sidecars. The intelligence layer (Ruflo, EverMemOS) can be built in parallel with the execution environment (Docker, git worktrees) because they have no shared dependencies. This parallelism is the key scheduling optimization for the roadmap.

The top risks are: (1) multi-agent coordination collapse where agents silently produce conflicting outputs, requiring a strict single-writer rule and hierarchical coordination protocol designed before any agent executes; (2) sidecar lifecycle failures on user machines where Python/Go processes crash, leak, or fail to start, requiring a Rust-side process supervisor with health checks and bundled single-file executables; (3) fork drift across 5 upstream repositories, mitigated by the adapter-over-fork pattern and monthly sync cadence; and (4) AGPL license contamination from MiroFish, which must be process-isolated when integrated in Phase 02. All four risks have well-documented mitigation strategies, but the agent coordination protocol is the highest-risk design decision in the entire project.

## Key Findings

### Recommended Stack

The stack is a Tauri 2.10 desktop shell with React 19.2 + TypeScript 6.0 on the frontend, Rust on the backend for performance-critical paths (Docker API via bollard, git operations via git2, process lifecycle management). The monorepo uses Turborepo + pnpm with Vite as the bundler and Biome for linting/formatting.

**Core technologies:**
- **Tauri 2.10:** Desktop shell with native OS access (file system, Docker, git) without Electron's memory overhead
- **React 19.2 + TypeScript 6.0:** UI framework, required by Monaco, React Flow, and xterm wrappers
- **Zustand 5 + TanStack Query 5:** Client state (panel layouts, agent status) and server state (cached Tauri command results)
- **Monaco Editor:** Code editor -- the only serious option for IDE-grade syntax highlighting, intellisense, diffing
- **React Flow 12.10:** Architecture canvas for agent workflow visualization
- **xterm.js 6:** Terminal emulator for agent output streams and build logs
- **Hono 4.12:** Ultrafast API sidecar compiled to single binary via @yao-pkg/pkg
- **Drizzle ORM 0.45:** Type-safe SQL in proxy mode (frontend computes SQL, Rust backend executes via sqlx)
- **bollard (Rust):** Docker API for agent container orchestration, type-safe, no CLI dependency
- **git2 (Rust):** Git worktree management for agent isolation, no PATH dependency on git CLI
- **Clerk:** Auth with Organizations model, but requires workarounds for Tauri WebView (no OAuth, email/password only)

**Critical version notes:** Monaco `@monaco-editor/react@next` is required for React 19 compatibility. Clerk in Tauri requires patching `globalThis.fetch` for the HTTP plugin. Drizzle requires the proxy pattern since `readMigrationFiles()` uses Node's `fs`.

### Expected Features

**Must have (table stakes):**
- Natural language to code generation (T1)
- Multi-file editing/generation via agent swarm (T2)
- Codebase context awareness with indexing/embedding (T3)
- Git integration with bidirectional GitHub sync, agent commits, PRs (T5)
- Error detection and auto-fix loop (T6)
- Model flexibility across OpenAI/Anthropic/Gemini (T7)
- Terminal/command execution in sandboxed containers (T8)
- File explorer (T9)
- Sandboxed execution via Scion/Docker (T12)

**Should have (differentiators for M1):**
- Connected Studios pipeline: PRD to Plan to Build (D1) -- core product identity
- Multi-agent swarm with specialized roles (D2) -- competitive moat
- Persistent cross-session memory via EverMemOS (D4) -- far beyond competitor memory
- Security-by-default via Guidance Control Plane (D6) -- enterprise-grade from day one
- Secrets management with encrypted vault (D7) -- practical necessity for agent execution
- Agent traceability via structured commits (D8) -- trust and debugging
- Local-first + offline capable (D10) -- natural from Tauri architecture

**Defer to Phase 02+:**
- Source Discovery and Qualification (D3) -- high value but high risk for M1 scope
- Architecture Synthesis engine (D5) -- depends on Source Discovery
- Commercialization Studio (D9) -- auth/billing/multi-tenancy pipeline stages
- Real-time multi-user collaboration -- design for it, do not build it
- Live preview (T4) -- desirable but deferrable if timeline is tight
- Plugin marketplace -- premature, zero users

**Explicit anti-features:** Browser-based IDE, simple code autocomplete, generic chat assistant, vibe coding.

### Architecture Approach

Hub-and-spoke architecture where the Tauri Rust core is the single hub managing all process lifecycles, state, and IPC routing. The React frontend, Python/Go sidecars, and Docker containers are spokes that communicate exclusively through the hub. The frontend NEVER communicates directly with sidecars. Three-tier IPC: Commands for request/response, Channels for streaming (agent output -- this is critical, Events are explicitly not designed for high throughput), Events for lifecycle notifications.

**Major components:**
1. **Tauri Rust Core** -- Process lifecycle, state management (Mutex-wrapped AppState), IPC routing, security enforcement
2. **Sidecar Manager (Rust)** -- Supervises DeerFlow (Python/FastAPI) and Scion (Go) with health checks, restart, graceful shutdown
3. **Docker Manager (Rust via bollard)** -- Container CRUD, image management, resource limits, log streaming to Channels
4. **Git Manager (Rust via git2)** -- Worktree creation/cleanup per agent, branch management, merge orchestration
5. **Flow Engine (TypeScript)** -- Unified adapter over Ruflo (direct TS import) + DeerFlow (HTTP client) + Scion (HTTP client) + EverMemOS (direct TS import)
6. **React Frontend** -- Studio views, multiplexed agent live feed, file explorer, terminal emulator, Zustand stores + React Query

### Critical Pitfalls

1. **Multi-agent coordination collapse** -- Agents silently produce conflicting outputs with no error signals. Prevent with single-writer rule per file/resource, hierarchical coordination (Architect agent is authoritative), and explicit state machine for agent transitions. Design the protocol before any agent executes. This is the highest-risk design decision in the project.

2. **Sidecar lifecycle hell** -- DeerFlow/Scion crash silently, leak as orphan processes, fail on user machines due to missing dependencies. Prevent with Rust-side process supervisor (health checks every 5s, exponential backoff restart, graceful shutdown escalation), bundled single-file executables, and Docker dependency check on first launch.

3. **Fork drift across 5 upstream repos** -- Forks diverge within 3-6 months, making sync exponentially harder. Prevent with adapter-over-fork pattern, monthly sync cadence, pinned upstream releases, and upstream contributions where possible.

4. **AGPL-3.0 contamination from MiroFish** -- Viral copyleft license could require open-sourcing the entire product. MiroFish must run as process-isolated sidecar only (no code linking). Legal counsel required before Phase 02 integration. Audit ALL fork dependency trees for transitive AGPL in Phase 1.

5. **WebView divergence across platforms** -- WebKit GTK on Linux is actively regressing; features that work on macOS break silently. Prevent with cross-platform CI from Phase 1, ES2021 baseline, and explicit testing of SSE/WebSocket streaming on all platforms.

## Implications for Roadmap

Based on combined research, the architecture has a clear dependency chain that dictates phase ordering. The intelligence layer (Ruflo, EverMemOS) and the execution environment (Docker, git worktrees) can be built in parallel, which is the key scheduling optimization.

### Phase 1: Foundation and Shell
**Rationale:** Everything depends on the Tauri shell, Rust module structure, IPC layer, and database. No agent, studio, or sidecar can function without this foundation.
**Delivers:** Working Tauri desktop app with React scaffold, Zustand + React Query state, SQLite persistence via Drizzle proxy pattern, cross-platform CI pipeline, monorepo structure with Turborepo + pnpm, Rust module structure (commands/, sidecar/, docker/, git/).
**Addresses:** T9 (file explorer scaffold), D10 (local-first architecture)
**Avoids:** Pitfall 1 (WebView divergence -- cross-platform CI from day 1), Pitfall 7 (schema drift -- UUID keys and shared schema source from the start), Pitfall 10 (auto-update key loss -- secure signing key before first release)

### Phase 2: Sidecar Infrastructure and Process Management
**Rationale:** Agents cannot execute without DeerFlow and Scion running as supervised processes. This is the hardest infrastructure layer and the one most likely to cause production failures if under-engineered.
**Delivers:** Rust-side Sidecar Manager with health checks, restart, graceful shutdown. DeerFlow bundled as PyInstaller binary. Scion bundled as Go binary. Both declared as Tauri externalBin. Docker dependency detection and onboarding flow. Unified structured logging with trace ID propagation.
**Addresses:** T8 (terminal/command execution infrastructure), T12 (sandboxed execution infrastructure)
**Avoids:** Pitfall 5 (sidecar lifecycle hell), Pitfall 9 (Docker Desktop dependency), Pitfall 12 (polyglot observability blind spots)

### Phase 3: Execution Environment (Docker + Git Worktrees)
**Rationale:** Depends on sidecar infrastructure being stable. Docker containers and git worktrees are the physical isolation layer for agents.
**Delivers:** Docker Manager via bollard (container CRUD, resource limits, log streaming). Git Manager via git2 (worktree creation/cleanup, branch management). Container-to-worktree binding. Agent output streaming pipeline (Container -> Scion -> Rust -> Channel -> React).
**Addresses:** T12 (sandboxed execution complete), D8 (agent traceability via structured commits)
**Avoids:** Pitfall 6 (worktree explosion -- single-writer rule, worktree cap at 3-5 concurrent), Pitfall 8 (WebView memory pressure -- virtualized rendering from day 1, ring buffers per agent)

### Phase 4: Intelligence Layer (parallel with Phase 3)
**Rationale:** Ruflo and EverMemOS are TypeScript packages with no Docker/git dependency. Can be built simultaneously with Phase 3 for schedule compression. This is the main parallelism opportunity.
**Delivers:** Flow Engine unified interface. Ruflo adapter (intelligence routing, policy enforcement). EverMemOS adapter (memory cells, recall). Task graph planning. Agent routing decisions. Coordination protocol design (single-writer, hierarchical authority, state machine transitions).
**Addresses:** T7 (model flexibility), D4 (persistent cross-session memory), D6 (security-by-default via Guidance Control Plane)
**Avoids:** Pitfall 2 (agent coordination collapse -- design coordination protocol here before agents execute), Pitfall 3 (fork drift -- adapter-over-fork pattern established here)

### Phase 5: Studios (PRD, Planning, Build and Test)
**Rationale:** Studios are the user-facing value layer. They require all infrastructure (Phases 1-4) to function. This is where the product becomes usable and differentiates from competitors.
**Delivers:** Studio I (PRD Wizard -- structured intent capture). Studio III (Planning -- GitHub connection, build plan generation). Studio VI (Build and Test -- agent live feed, file explorer, terminal, test runner). Core agent swarm (Architect, Builder, Security, QA). Error detection and auto-fix loop. Git integration (commits, PRs).
**Addresses:** T1 (NL to code), T2 (multi-file editing), T3 (codebase context), T5 (git integration), T6 (error detection), D1 (connected Studios pipeline), D2 (multi-agent swarm), D7 (secrets management)
**Avoids:** Pitfall 2 (uses coordination protocol from Phase 4)

### Phase 6: Auth, Polish, and Distribution
**Rationale:** Auth, onboarding, and distribution are required for release but do not affect core functionality. Building them last avoids premature optimization and keeps Clerk workarounds isolated.
**Delivers:** Clerk integration (email/password with Tauri workarounds). First-launch onboarding flow (Docker check, system requirements). Auto-update infrastructure. EV code signing for Windows. Cross-platform release builds.
**Addresses:** T11 (authentication)
**Avoids:** Pitfall 11 (SmartScreen -- EV certificate procured 4 weeks before release)

### Phase Ordering Rationale

- **Foundation first:** Every component depends on Tauri shell, IPC, and persistence. No parallelization possible here.
- **Sidecar infrastructure before execution environment:** Scion and DeerFlow must be running before Docker containers can be orchestrated through them.
- **Intelligence layer parallel with execution environment:** Ruflo and EverMemOS have zero dependency on Docker/git. This is the main schedule compression opportunity and should be exploited.
- **Studios last in the core pipeline:** Studios are consumers of all infrastructure layers. Building them first would mean stubbing everything below, which produces skeleton code.
- **Auth and distribution deferred to polish phase:** Clerk's Tauri workarounds are non-trivial but do not block core development. Distribution concerns are release-time, not dev-time.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 2 (Sidecar Infrastructure):** PyInstaller bundling for DeerFlow is finicky (hidden imports, dynamic deps). The Rust process supervisor pattern has community examples but no official Tauri plugin -- expect custom implementation work.
- **Phase 4 (Intelligence Layer):** Ruflo and EverMemOS API surfaces are project-specific forks. Their actual TypeScript interfaces need validation against current fork state. The agent coordination protocol (single-writer, hierarchical Architect authority, state machine transitions) is the highest-risk design decision and needs dedicated research.
- **Phase 5 (Studios):** The PRD-to-code pipeline is Ultrabuilder's core differentiator with no direct precedent to copy. Studio UX design will require iteration.

Phases with standard patterns (skip deep research):
- **Phase 1 (Foundation):** Tauri + React + Zustand + SQLite is extremely well-documented. Drizzle proxy pattern has a published tutorial.
- **Phase 3 (Execution Environment):** bollard and git2 are mature Rust crates with extensive docs. Docker container management and git worktree operations are well-understood patterns.
- **Phase 6 (Auth and Distribution):** Clerk integration has a community plugin. Tauri auto-update and code signing are documented in official guides.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All technologies are mature, actively maintained, and have official documentation. Only Clerk-in-Tauri is MEDIUM due to community-maintained plugin. |
| Features | HIGH | Based on analysis of 8 reference products with official feature documentation. Competitive positioning is clear and well-supported. |
| Architecture | HIGH (structure), MEDIUM (subsystem APIs) | Hub-and-spoke with Tauri Rust core is community standard. IPC mechanisms directly from official docs. Flow Engine adapter interfaces need validation against actual fork APIs. |
| Pitfalls | HIGH | All critical pitfalls backed by multiple sources (Tauri GitHub issues, academic papers, legal precedent). Mitigation strategies are concrete. |

**Overall confidence:** HIGH

### Gaps to Address

- **Clerk + Tauri integration depth:** Community plugin production readiness is unverified. Validate early in Phase 6 or have a fallback auth strategy (custom JWT + lightweight auth server).
- **DeerFlow and Scion actual API surfaces:** Research assumes HTTP/REST interfaces from upstream README. Actual API contracts of the forked versions need validation when forks are set up.
- **Ruflo and EverMemOS TypeScript interfaces:** Consumed as direct imports. Current exported API surface in the fork needs audit before Flow Engine adapter design is finalized.
- **Agent coordination protocol specifics:** Research identifies the need (single-writer, hierarchical, state machine) but the exact protocol design (message formats, conflict resolution rules, rollback semantics) requires dedicated design work in Phase 4 planning.
- **Resource requirements for full agent swarm:** Architecture research estimates 12GB+ RAM for 12 agents. Actual resource profiling needed during Phase 3 to set realistic container limits and minimum system requirements.
- **MiroFish AGPL transitive dependency audit:** Phase 1 should include a license audit of ALL fork dependency trees, not just MiroFish. Transitive AGPL contamination in DeerFlow or Scion would be equally problematic.
- **Monaco worker configuration in Tauri WebView:** Known to require special setup for web worker paths. Needs a spike early in Phase 1 or Phase 5.
- **Cross-platform Docker availability:** Docker Desktop is not free for large enterprises. Need to document Docker as a prerequisite and consider Podman as an alternative.

## Sources

### Primary (HIGH confidence)
- Tauri 2.0 official documentation -- IPC, state management, shell plugin, sidecar embedding, distribution
- React 19.2, TypeScript 6.0, Tailwind CSS 4, Zustand 5, TanStack Query 5 official docs
- bollard (Rust Docker SDK), git2 (libgit2 bindings) crate documentation
- Cursor, Windsurf, Devin, GitHub Copilot, Bolt, Lovable, Replit Agent, v0 official feature documentation
- Multi-agent coordination failure analysis (Towards Data Science, Galileo)
- AGPL license implications (Open Core Ventures, Google AGPL policy)

### Secondary (MEDIUM confidence)
- tauri-plugin-clerk community plugin for Clerk + Tauri integration
- Drizzle + Tauri proxy pattern tutorial (dev.to)
- tauri-sidecar-manager community plugin for lifecycle patterns
- Fork drift case studies (Preset blog)
- PowerSync/ElectricSQL for SQLite-Postgres sync patterns
- Git worktree patterns for AI agents (Termdock, Vibehackers, Pochi)

### Tertiary (LOW confidence)
- react-xtermjs wrapper -- limited community validation, only actively maintained React wrapper for xterm.js
- @yao-pkg/pkg for Hono sidecar compilation -- community fork of vercel/pkg, functional but niche
- DeerFlow/Scion specific API surface details -- based on upstream README, not validated against fork state

---
*Research completed: 2026-03-28*
*Ready for roadmap: yes*
