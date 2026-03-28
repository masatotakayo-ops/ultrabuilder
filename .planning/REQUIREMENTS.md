# Requirements: Ultrabuilder

**Defined:** 2026-03-28
**Core Value:** User describes what they want → system plans it → agents build it into a GitHub repo with real, working code.

## v1 Requirements

Requirements for Milestone 1 (Foundation → Core Pipeline). Each maps to roadmap phases.

### Foundation

- [ ] **FNDN-01**: Tauri 2.0 desktop app launches on macOS, Windows, and Linux with React UI
- [ ] **FNDN-02**: Turborepo monorepo with apps/desktop and packages/ structure builds successfully
- [ ] **FNDN-03**: SQLite database persists project state locally via Drizzle ORM proxy pattern
- [ ] **FNDN-04**: Cross-platform CI pipeline runs on all 3 OS targets
- [ ] **FNDN-05**: Tauri IPC layer supports Commands (request/response), Channels (streaming), and Events (lifecycle)

### Sidecar Infrastructure

- [ ] **SIDE-01**: DeerFlow Python service starts as a supervised sidecar process from Tauri
- [ ] **SIDE-02**: Scion Go service starts as a supervised sidecar process from Tauri
- [ ] **SIDE-03**: Rust-side process supervisor performs health checks, auto-restart with exponential backoff, and graceful shutdown
- [ ] **SIDE-04**: DeerFlow is bundled as a single-file executable (PyInstaller) declared as Tauri externalBin
- [ ] **SIDE-05**: Scion is bundled as a single-file Go binary declared as Tauri externalBin
- [ ] **SIDE-06**: First-launch onboarding detects Docker availability and guides user through setup

### Execution Environment

- [ ] **EXEC-01**: Docker containers are created, started, stopped, and removed via bollard Rust crate
- [ ] **EXEC-02**: Agent code executes inside isolated Docker containers with resource limits (CPU, memory)
- [ ] **EXEC-03**: Git worktrees are created per agent via git2 Rust crate with dedicated feature branches
- [ ] **EXEC-04**: Agent output streams from Docker container through Scion to React UI via Tauri Channels in real time
- [ ] **EXEC-05**: Worktree cleanup runs automatically when agent tasks complete
- [ ] **EXEC-06**: Single-writer rule enforced — no two agents can write to the same file simultaneously

### Intelligence Layer

- [ ] **INTL-01**: Flow Engine provides a unified TypeScript interface over Ruflo, DeerFlow, Scion, and EverMemOS
- [ ] **INTL-02**: Ruflo adapter routes tasks to the best-suited agent using neural routing; routing passes file-type context (TypeScript / Python / Rust / YAML) to the Builder agent so it receives language-specific instruction profiles
- [ ] **INTL-03**: Ruflo's Guidance Control Plane enforces 7-layer policy gates that agents cannot bypass
- [ ] **INTL-04**: EverMemOS adapter stores and recalls structured memory (MemCells) across sessions; HNSW index must use quantized vector storage (PQ/SQ compression) to reduce MemCell memory footprint and speed similarity recall — validate configuration preserves recall accuracy (target ≥ 90%)
- [ ] **INTL-05**: Agent coordination protocol enforces hierarchical authority (Architect agent is authoritative) with explicit state machine transitions
- [ ] **INTL-06**: Model flexibility — user can select between OpenAI, Anthropic, and Google models for agent execution
- [ ] **INTL-07**: Each swarm agent has an explicit capability manifest defining allowed tools, write access scope, and prohibited actions — Architect: read-all + write architecture docs only; Builder: write code + run tests, no merge-to-main; Security: read-all + run scanners, no write code; QA: write tests + run suite, no modify source

### Studio I — Product Requirements

- [ ] **PRD-01**: User completes a wizard-driven flow to define product vision, personas, and feature requirements
- [ ] **PRD-02**: Wizard output is a structured PRD document (.md) saved to the project
- [ ] **PRD-03**: PRD captures: product description, target users, core features, constraints, and success criteria
- [ ] **PRD-04**: User can edit and re-run the PRD wizard at any time to update requirements

### Studio III — Planning

- [ ] **PLAN-01**: User connects a GitHub repository as the project's single source of truth
- [ ] **PLAN-02**: System verifies GitHub connection and confirms read/write access
- [ ] **PLAN-03**: Build plan is generated from the PRD — breaking features into ordered tasks for agents
- [ ] **PLAN-04**: User can review and approve the build plan before execution begins
- [ ] **PLAN-05**: Secrets (API keys, credentials, tokens) are stored in an encrypted vault scoped to the project
- [ ] **PLAN-06**: Secrets are injected into agent execution environments automatically without mid-build prompts

### Studio VI — Build & Test

- [ ] **BILD-01**: IDE-like file explorer shows the project's file tree in real time as agents generate code
- [ ] **BILD-02**: Agent live feed displays real-time output from all active agents with per-agent filtering
- [ ] **BILD-03**: Every agent action commits to the connected GitHub repo with structured commit messages
- [ ] **BILD-04**: User can pause, retry, or override any agent at checkpoints
- [ ] **BILD-05**: Integrated terminal displays build output and allows manual command execution
- [ ] **BILD-06**: Test runner executes unit and integration tests and reports results inline
- [ ] **BILD-07**: Error detection loop: agents detect test/build failures and auto-fix without user intervention
- [ ] **BILD-08**: Code editor (Monaco) displays generated code with syntax highlighting and diff view
- [ ] **BILD-09**: Architect agent creates and maintains CONVENTIONS.md in the target repo at each checkpoint — captures naming patterns, framework choices, testing conventions; Builder agent reads CONVENTIONS.md before every code generation task

### Agent Swarm

- [ ] **AGNT-01**: Architect agent creates the target architecture from the PRD and build plan
- [ ] **AGNT-02**: Builder agent generates code guided by the architecture and policy gates
- [ ] **AGNT-03**: Security agent continuously scans generated code for vulnerabilities
- [ ] **AGNT-04**: QA agent generates and executes tests, producing a build confidence score
- [ ] **AGNT-05**: Agents run in parallel where task dependencies allow
- [ ] **AGNT-06**: Agent traceability — every agent decision is logged with full context in git history

### GitHub Integration

- [ ] **GIT-01**: GitHub repo is connected via personal access token or GitHub App
- [ ] **GIT-02**: All agent commits use structured messages: `[agent-name] action: description`
- [ ] **GIT-03**: Completed features are merged to main branch via automated PR creation
- [ ] **GIT-04**: Branch protection rules and existing governance are respected — agents work within, not around

### Authentication

- [ ] **AUTH-01**: User can sign up and log in with email and password via Clerk
- [ ] **AUTH-02**: User session persists across app restarts
- [ ] **AUTH-03**: Clerk Organizations model is initialized (single-user for M1, team-ready architecture)

## v2 Requirements

Deferred to Milestone 2+. Tracked but not in current roadmap.

### Source Discovery & Qualification

- **DISC-01**: System searches GitHub, npm, PyPI, and other ecosystems for relevant packages
- **DISC-02**: Each candidate is scored across 7 weighted dimensions (feature fit, maintenance, quality, docs, security, ecosystem, compatibility)
- **DISC-03**: User reviews qualified sources and approves selections before synthesis

### Architecture Synthesis

- **ARCH-01**: System assigns sources to architectural layers (keep/replace/wrap/merge/discard)
- **ARCH-02**: Interactive architecture canvas visualizes module relationships and conflict zones

### Commercialization Studio

- **COMM-01**: Auth, billing, multi-tenancy added as pipeline stages to generated apps
- **COMM-02**: MiroFish viability predictor simulates market behavior before code is written

### Team Collaboration

- **TEAM-01**: Multiple team members access the same project with live presence indicators
- **TEAM-02**: Role-based access (Owner, Architect, Builder, Reviewer, Observer)
- **TEAM-03**: Review flows allow designated reviewers to approve generated code

### Integrations

- **INTG-01**: Slack integration — live build feed, checkpoint approvals, deployment notifications
- **INTG-02**: Linear integration — agent tasks as issues, two-way sync, build triggers
- **INTG-03**: Multi-channel agent input (Telegram, WhatsApp, Feishu)

### Additional Studios

- **STUD-01**: Architecture Studio — interactive canvas for architecture visualization and editing
- **STUD-02**: Interface Design Studio — theme-aware UI design with live preview
- **STUD-03**: Production Deployment Studio — deploy pipeline, environment management, release tagging
- **STUD-04**: Commercialization Studio — business model, pricing strategy, GTM plan

### Theme Engine

- **THEM-01**: Theme creation via visual editor, text prompt, or URL extraction
- **THEM-02**: Team Theme Library with versioned .json packages

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Browser-based IDE | Desktop is the product lane — competing with Bolt/Lovable is a distraction |
| Simple code autocomplete | Factory, not typing assistant — Cursor/Copilot own this |
| Generic chat assistant | Commoditized, zero differentiation — use studio-specific agents instead |
| Vibe coding / prompt-and-pray | Structured pipeline is the anti-vibe-coding differentiator |
| Plugin marketplace (M1) | Zero users, zero plugins — premature infrastructure |
| Real-time multi-user editing (M1) | Massive engineering (CRDTs), single-user first |
| Hosting infrastructure | Push to GitHub, users deploy via their CI/CD |
| Language-specific fine-tuned models | Use frontier models, focus on orchestration |
| Mobile app | Desktop-first, no mobile use case for IDE |
| Lego Cube Component Catalog (M1) | Defer until Source Discovery is built |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| FNDN-01 | Phase 1 | Pending |
| FNDN-02 | Phase 1 | Pending |
| FNDN-03 | Phase 1 | Pending |
| FNDN-04 | Phase 1 | Pending |
| FNDN-05 | Phase 1 | Pending |
| SIDE-01 | Phase 2 | Pending |
| SIDE-02 | Phase 2 | Pending |
| SIDE-03 | Phase 2 | Pending |
| SIDE-04 | Phase 2 | Pending |
| SIDE-05 | Phase 2 | Pending |
| SIDE-06 | Phase 2 | Pending |
| EXEC-01 | Phase 3 | Pending |
| EXEC-02 | Phase 3 | Pending |
| EXEC-03 | Phase 3 | Pending |
| EXEC-04 | Phase 3 | Pending |
| EXEC-05 | Phase 3 | Pending |
| EXEC-06 | Phase 3 | Pending |
| INTL-01 | Phase 4 | Pending |
| INTL-02 | Phase 4 | Pending |
| INTL-03 | Phase 4 | Pending |
| INTL-04 | Phase 4 | Pending |
| INTL-05 | Phase 4 | Pending |
| INTL-06 | Phase 4 | Pending |
| INTL-07 | Phase 4 | Pending |
| PRD-01 | Phase 5 | Pending |
| PRD-02 | Phase 5 | Pending |
| PRD-03 | Phase 5 | Pending |
| PRD-04 | Phase 5 | Pending |
| PLAN-01 | Phase 5 | Pending |
| PLAN-02 | Phase 5 | Pending |
| PLAN-03 | Phase 5 | Pending |
| PLAN-04 | Phase 5 | Pending |
| PLAN-05 | Phase 5 | Pending |
| PLAN-06 | Phase 5 | Pending |
| BILD-01 | Phase 5 | Pending |
| BILD-02 | Phase 5 | Pending |
| BILD-03 | Phase 5 | Pending |
| BILD-04 | Phase 5 | Pending |
| BILD-05 | Phase 5 | Pending |
| BILD-06 | Phase 5 | Pending |
| BILD-07 | Phase 5 | Pending |
| BILD-08 | Phase 5 | Pending |
| BILD-09 | Phase 5 | Pending |
| AGNT-01 | Phase 5 | Pending |
| AGNT-02 | Phase 5 | Pending |
| AGNT-03 | Phase 5 | Pending |
| AGNT-04 | Phase 5 | Pending |
| AGNT-05 | Phase 5 | Pending |
| AGNT-06 | Phase 5 | Pending |
| GIT-01 | Phase 5 | Pending |
| GIT-02 | Phase 5 | Pending |
| GIT-03 | Phase 5 | Pending |
| GIT-04 | Phase 5 | Pending |
| AUTH-01 | Phase 6 | Pending |
| AUTH-02 | Phase 6 | Pending |
| AUTH-03 | Phase 6 | Pending |

**Coverage:**
- v1 requirements: 56 total
- Mapped to phases: 56
- Unmapped: 0

---
*Requirements defined: 2026-03-28*
*Last updated: 2026-03-28 after roadmap creation*
