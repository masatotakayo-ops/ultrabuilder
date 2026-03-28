# Roadmap: Ultrabuilder

## Overview

Ultrabuilder delivers an autonomous software factory as a Tauri desktop app. The roadmap builds infrastructure bottom-up: desktop shell and persistence first, then sidecar process management, then the execution environment (Docker/git) and intelligence layer (Ruflo/EverMemOS) in parallel, then the user-facing Studios that consume all infrastructure, and finally auth and distribution for release readiness. The golden path -- user describes what they want, system plans it, agents build it into a GitHub repo -- is fully operational at the end of Phase 5.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundation and Shell** - Tauri desktop app with React UI, monorepo, SQLite persistence, IPC layer, and cross-platform CI
- [ ] **Phase 2: Sidecar Infrastructure** - DeerFlow and Scion as supervised sidecar processes with health checks, bundled binaries, and Docker onboarding
- [ ] **Phase 3: Execution Environment** - Docker container orchestration and git worktree management for isolated agent execution
- [ ] **Phase 4: Intelligence Layer** - Unified Flow Engine over Ruflo, DeerFlow, Scion, and EverMemOS with agent coordination protocol (parallel with Phase 3)
- [ ] **Phase 5: Connected Studios** - PRD wizard, planning with GitHub integration, and IDE-like build environment with agent swarm
- [ ] **Phase 6: Auth and Distribution** - Clerk authentication, session persistence, and Organizations model for release readiness

## Phase Details

### Phase 1: Foundation and Shell
**Goal**: A working Tauri desktop application launches cross-platform with React UI, local persistence, and the monorepo build pipeline
**Depends on**: Nothing (first phase)
**Requirements**: FNDN-01, FNDN-02, FNDN-03, FNDN-04, FNDN-05
**Success Criteria** (what must be TRUE):
  1. User can launch the desktop app on macOS, Windows, and Linux and see the React UI
  2. App persists project data locally across restarts via SQLite
  3. Monorepo builds all packages successfully with a single command
  4. CI pipeline runs and passes on all three OS targets
  5. Frontend can invoke Rust commands, receive streaming data via Channels, and react to lifecycle Events
**Plans**: 5 plans

Plans:
- [ ] 01-01-PLAN.md -- Monorepo scaffold (Turborepo, pnpm, packages/shared, Biome)
- [ ] 01-02-PLAN.md -- Tauri app + React shell (VS Code layout, titlebar, menu, router, theme, fonts)
- [ ] 01-03-PLAN.md -- SQLite + Drizzle proxy + Projects CRUD (schema, migrations, run_sql, UI)
- [ ] 01-04-PLAN.md -- IPC layer proof (Channels log streaming, Events lifecycle, IPC debug panel)
- [ ] 01-05-PLAN.md -- CI pipeline (GitHub Actions matrix, Vitest setup, Tauri IPC mocks)

### Phase 2: Sidecar Infrastructure
**Goal**: DeerFlow and Scion run as reliable, supervised sidecar processes that start with the app, recover from crashes, and shut down cleanly
**Depends on**: Phase 1
**Requirements**: SIDE-01, SIDE-02, SIDE-03, SIDE-04, SIDE-05, SIDE-06
**Success Criteria** (what must be TRUE):
  1. DeerFlow Python service starts automatically when the app launches and responds to health checks
  2. Scion Go service starts automatically when the app launches and responds to health checks
  3. If either sidecar crashes, the supervisor restarts it with exponential backoff without user intervention
  4. Both sidecars are bundled as single-file executables requiring no external Python/Go installation
  5. On first launch, the app detects Docker availability and guides the user through setup if missing
**Plans**: TBD

Plans:
- [ ] 02-01: TBD
- [ ] 02-02: TBD
- [ ] 02-03: TBD

### Phase 3: Execution Environment
**Goal**: Agents can execute code in isolated Docker containers with dedicated git worktrees, streaming output to the UI in real time
**Depends on**: Phase 2
**Requirements**: EXEC-01, EXEC-02, EXEC-03, EXEC-04, EXEC-05, EXEC-06
**Success Criteria** (what must be TRUE):
  1. Docker containers are created and destroyed programmatically with CPU and memory resource limits enforced
  2. Agent code runs inside an isolated container and its output streams to the React UI in real time
  3. Each agent gets a dedicated git worktree on a feature branch, cleaned up automatically on task completion
  4. No two agents can write to the same file simultaneously (single-writer rule enforced)
**Plans**: TBD

Plans:
- [ ] 03-01: TBD
- [ ] 03-02: TBD
- [ ] 03-03: TBD

### Phase 4: Intelligence Layer
**Goal**: A unified Flow Engine routes tasks to agents, enforces policy gates, persists memory across sessions, and coordinates agent behavior through an authoritative protocol
**Depends on**: Phase 1 (parallel with Phase 3 -- no Docker/git dependency)
**Requirements**: INTL-01, INTL-02, INTL-03, INTL-04, INTL-05, INTL-06, INTL-07
**Success Criteria** (what must be TRUE):
  1. Flow Engine exposes a single TypeScript interface that abstracts Ruflo, DeerFlow, Scion, and EverMemOS
  2. Tasks are routed to the best-suited agent via Ruflo's neural routing, and the user can select between OpenAI, Anthropic, and Google models
  3. Policy gates block agent actions that violate security rules -- agents cannot bypass the Guidance Control Plane
  4. Memory persists across sessions -- the system recalls prior project context when reopened
  5. Agent coordination follows a hierarchical protocol where the Architect agent is authoritative and state transitions are explicit
**Plans**: TBD

Plans:
- [ ] 04-01: TBD
- [ ] 04-02: TBD
- [ ] 04-03: TBD

### Phase 5: Connected Studios
**Goal**: Users can describe what they want to build, the system plans it, and agents build it into a connected GitHub repo with real working code -- the complete golden path
**Depends on**: Phase 3, Phase 4
**Requirements**: PRD-01, PRD-02, PRD-03, PRD-04, PLAN-01, PLAN-02, PLAN-03, PLAN-04, PLAN-05, PLAN-06, BILD-01, BILD-02, BILD-03, BILD-04, BILD-05, BILD-06, BILD-07, BILD-08, BILD-09, AGNT-01, AGNT-02, AGNT-03, AGNT-04, AGNT-05, AGNT-06, GIT-01, GIT-02, GIT-03, GIT-04
**Success Criteria** (what must be TRUE):
  1. User completes the PRD wizard and produces a structured requirements document that can be edited and re-run
  2. User connects a GitHub repo, the system verifies access, and a build plan is generated from the PRD for user review and approval
  3. Agents (Architect, Builder, Security, QA) execute the plan in parallel where dependencies allow, with every action committed to GitHub using structured messages
  4. User sees real-time agent output, file tree updates, code diffs, build output, and test results in an IDE-like environment
  5. User can pause, retry, or override any agent at checkpoints, and agents auto-fix test/build failures without user intervention
**Plans**: TBD

Plans:
- [ ] 05-01: TBD
- [ ] 05-02: TBD
- [ ] 05-03: TBD
- [ ] 05-04: TBD
- [ ] 05-05: TBD

### Phase 6: Auth and Distribution
**Goal**: Users can securely access their accounts and the app is ready for release distribution
**Depends on**: Phase 5
**Requirements**: AUTH-01, AUTH-02, AUTH-03
**Success Criteria** (what must be TRUE):
  1. User can sign up and log in with email and password via Clerk
  2. User session persists across app restarts without re-authentication
  3. Clerk Organizations model is initialized and functional for single-user use with team-ready architecture
**Plans**: TBD

Plans:
- [ ] 06-01: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3 (parallel with 4) -> 5 -> 6
Note: Phase 3 and Phase 4 can execute in parallel (intelligence layer has no Docker/git dependency).

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation and Shell | 0/5 | Planning complete | - |
| 2. Sidecar Infrastructure | 0/3 | Not started | - |
| 3. Execution Environment | 0/3 | Not started | - |
| 4. Intelligence Layer | 0/3 | Not started | - |
| 5. Connected Studios | 0/5 | Not started | - |
| 6. Auth and Distribution | 0/1 | Not started | - |
