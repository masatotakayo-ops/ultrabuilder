# Ultrabuilder

## What This Is

Ultrabuilder is an autonomous software factory — a Tauri desktop application that turns product intent into production-ready commercial SaaS through wizard-driven Connected Studios, a coordinated agent swarm, and deep GitHub integration. Users define what they want to build, and the platform's Flow Engine (Ruflo + DeerFlow + Scion + EverMemOS) plans, generates, tests, and deploys the software into a GitHub repository. Built for developers and teams who want AI-powered software synthesis with full traceability.

## Core Value

The core pipeline must work: a user describes what they want → the system plans it → agents build it into a GitHub repo with real, working code. Everything else is secondary to this golden path.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Product Requirements Studio (Studio I) — wizard-driven PRD creation
- [ ] Planning Studio (Studio III) — GitHub repo connection, secrets audit, build plan
- [ ] Build & Test Studio (Studio VI) — IDE-like environment with agent live feed, file explorer, integrated test runner
- [ ] Flow Engine integration — Ruflo intelligence routing + DeerFlow sandboxed execution + Scion container isolation + EverMemOS memory
- [ ] GitHub integration — mandatory bidirectional, every agent action commits with structured messages
- [ ] Agent swarm — Architect, Discoverer, Builder, Security, QA agents (core subset of 12)
- [ ] Basic authentication — single-user login via Clerk, designed for future team expansion
- [ ] Secrets Management — encrypted vault, scoped at workspace/project/environment levels
- [ ] Source Discovery & Qualification — search ecosystems, score candidates, gate synthesis
- [ ] Architecture Synthesis — assign sources to layers, decide keep/replace/wrap/merge/discard

### Out of Scope

- Commercialization Studio + MiroFish — Phase 02, not needed for core pipeline
- Architecture Studio (interactive canvas) — Phase 02, nice-to-have after core works
- Interface Design Studio — Phase 02, depends on Theme Engine
- Production Deployment Studio — Phase 02, manual deploy sufficient initially
- Slack integration — Phase 02, not blocking core value
- Linear integration — Phase 02, not blocking core value
- Multi-channel agent input (Telegram, WhatsApp, Feishu) — Phase 02+
- Theme Engine — Phase 02, cosmetic layer
- Team collaboration (roles, concurrent access, review flows) — Phase 02, single-user first
- Enterprise features (SSO/SAML/SCIM, compliance) — Phase 03
- Marketplace — Phase 04

## Context

### Open-Source Foundations

The platform is built on 5 forked open-source systems, integrated as a unified Flow Engine:

| System | Role | Repo | Language | License |
|--------|------|------|----------|---------|
| Ruflo | Intelligence — routing, policy, learning | github.com/ruvnet/ruflo | TypeScript | Apache 2.0 |
| DeerFlow | Execution — Docker sandboxes, sub-agents | github.com/bytedance/deer-flow | Python | MIT |
| Scion | Isolation — containers, worktrees, lifecycle | github.com/GoogleCloudPlatform/scion | Go | Apache 2.0 |
| EverMemOS | Memory — MemCells, consolidation, recall | github.com/EverMind-AI/EverMemOS | Python | Apache 2.0 |
| MiroFish | Prediction — swarm simulation (Phase 02) | github.com/666ghj/MiroFish | Python | AGPL-3.0 |

### Architecture

Tauri 2.0 desktop app with Turborepo monorepo:

```
ultrabuilder/
├── apps/
│   └── desktop/              ← Tauri 2.0 shell + React UI
├── packages/
│   ├── ruflo/                ← Intelligence layer (forked, TS adapter)
│   ├── deerflow/             ← Execution harness (API client SDK)
│   ├── scion/                ← Container runtime (API client SDK)
│   ├── evermemos/            ← Memory OS (forked, TS adapter)
│   ├── flow-engine/          ← Unified adapter over all 4 systems
│   └── shared/               ← Types, constants, utilities
├── turbo.json
└── package.json
```

- DeerFlow (Python) and Scion (Go) run as sidecar services — the desktop app calls them via local API
- Ruflo (TypeScript) and EverMemOS are wrapped with TS adapters and consumed directly
- MiroFish deferred to Phase 02

### Key Stats from PRD

- 313 MCP tools pre-wired
- 84.8% SWE-Bench solve rate (Ruflo)
- 93% LoCoMo memory accuracy (EverMemOS)
- 12 named agents in the swarm
- 16 synthesis engines across 4 super-layers

## Constraints

- **Tech stack**: Tauri 2.0 (Rust + WebView), React + TypeScript, shadcn/ui + Tailwind, Turborepo monorepo — chosen for IDE-like requirements (file system, Docker, git, long-running agents)
- **Auth**: Clerk with Organizations from day one — single-user now, team-ready architecture
- **Database**: SQLite local + Postgres for team/cloud sync — local-first, sync when connected
- **Agent execution**: DeerFlow runs in Docker containers locally — no serverless timeout limits
- **Git**: Every agent action must commit to the connected GitHub repo with structured messages
- **Security**: Ruflo's 7-layer Guidance Control Plane enforces policy gates agents cannot bypass
- **Desktop distribution**: Tauri auto-updater for releases, cross-platform (macOS, Windows, Linux)

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Tauri desktop over Next.js web | IDE-like UX needs: file system, Docker, git, long-running agents, canvas editors. Every successful AI coding tool is desktop. | — Pending |
| Turborepo monorepo | Single repo for web UI + all forked packages. Shared types, incremental builds, one push deploys. | — Pending |
| Vertical slice milestone (Studios I→III→VI) | Core pipeline end-to-end before peripheral studios. Proves core value fastest. | — Pending |
| Clerk for auth | Vercel Marketplace native, Organizations model matches PRD role system (Owner/Architect/Builder/Reviewer/Observer). | — Pending |
| Local-first + cloud sync | Desktop app works offline, agents run locally zero latency, team sync via API when connected. Matches PRD's hosted + BYOC modes. | — Pending |
| DeerFlow/Scion as sidecar services | Python/Go repos can't be directly bundled in Tauri. Run as local API services, TS SDK wraps calls. | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-03-28 after initialization*
