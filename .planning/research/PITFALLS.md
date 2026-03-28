# Domain Pitfalls

**Domain:** Autonomous software factory / AI-powered IDE desktop app (Tauri + multi-agent orchestration)
**Researched:** 2026-03-28

---

## Critical Pitfalls

Mistakes that cause rewrites, multi-week delays, or architectural dead ends.

### Pitfall 1: WebView Divergence Across Platforms (Tauri 2.0)

**What goes wrong:** Tauri uses the OS-native WebView (WebKit on macOS/Linux, WebView2/Chromium on Windows). WebKit GTK on Linux is actively regressing -- basic features like video playback are broken, and stability has worsened with each release. CSS/JS features that work perfectly on macOS + Windows silently break on Linux.

**Why it happens:** Tauri chose native WebViews for binary size (no bundled Chromium), but WebKit GTK receives minimal investment from the Linux desktop community. Feature parity is a myth -- ES2021 is the safe baseline, but Web APIs (WebSockets, SSE, Web Workers, clipboard, drag-drop) behave differently per engine.

**Consequences:** Features work in dev (macOS) but break in CI/user machines (Linux). Real-time agent streaming via SSE/WebSocket is the core UX -- if it breaks on one platform, the product is unusable there. Late discovery means rewriting rendering or adding polyfills under pressure.

**Prevention:**
- CI must build and smoke-test on all three platforms from Phase 1. Not "later."
- Use a WebView feature compatibility matrix. Test SSE streaming, WebSocket connections, and DOM update performance on Linux WebKit explicitly.
- Limit CSS/JS to ES2021 baseline. Use feature detection, not assumption.
- Accept that Linux support may require a "best effort" label until WebKit GTK stabilizes.

**Detection:** Agent live feed renders correctly on macOS but shows blank panels, dropped frames, or frozen streams on Linux. CSS layouts shift on Windows vs macOS.

**Phase:** Phase 1 (Tauri shell setup). Set up cross-platform CI immediately. Do not defer.

**Confidence:** HIGH -- multiple Tauri community reports and official documentation confirm WebKit GTK instability.

---

### Pitfall 2: Multi-Agent Coordination Collapse ("Bag of Agents" Trap)

**What goes wrong:** Agents operating concurrently produce conflicting outputs, overwrite each other's state, or deadlock waiting for shared resources. Research shows coordination breakdowns account for ~37% of multi-agent system failures, and these failures generate no explicit error signals -- they silently corrupt output.

**Why it happens:** Developers treat multi-agent systems as "run N agents in parallel and merge results." Without explicit concurrency control, agents read stale state, write conflicting changes to the same files, and produce contradictory architectural decisions. The error rate compounds: studies show a "17x error trap" where naive multi-agent setups amplify errors rather than reducing them.

**Consequences:** The Builder agent generates code that contradicts the Architect agent's decisions. The Security agent and QA agent modify the same files simultaneously. The user sees "complete" output that is internally inconsistent. Debugging is extremely difficult because no single agent errored -- the system-level behavior is wrong.

**Prevention:**
- Single-writer rule: Each file/resource has exactly one owning agent at any time. Other agents request changes through the owner.
- Hierarchical coordination: The Architect agent is authoritative. Builder, Security, QA agents receive instructions, not autonomy over shared state.
- Explicit state machine: Agent transitions (planning -> building -> testing -> reviewing) are sequential gates, not concurrent free-for-all.
- Conflict detection before merge: Use `git merge-tree` (dry-run merge) between agent worktrees before actual merge. Tools like Clash exist for this.

**Detection:** Integration tests produce different results on repeated runs. Merged code has contradictory patterns (e.g., two auth systems, conflicting API schemas). Agent logs show concurrent writes to the same file paths.

**Phase:** Phase 1 (Flow Engine design). The coordination protocol must be designed before any agent executes. Retrofitting coordination onto a "bag of agents" requires a rewrite.

**Confidence:** HIGH -- multiple production post-mortems and research papers confirm this failure mode.

---

### Pitfall 3: Fork Drift Across 5 Upstream Repositories

**What goes wrong:** The 5 forked repos (Ruflo, DeerFlow, Scion, EverMemOS, MiroFish) diverge from upstream. Within 3-6 months, upstream makes breaking changes to APIs, file structures, and dependencies. Your fork accumulates parallel modifications. Rebasing becomes exponentially harder with each upstream release. Eventually, syncing is abandoned and you own 5 independent codebases.

**Why it happens:** Forking feels free at the start. The cost is deferred -- it compounds over time. Each fork requires someone to monitor upstream releases, evaluate breaking changes, resolve conflicts, and test integration. With 5 forks across 3 languages (TypeScript, Python, Go), this is effectively 5x the maintenance burden.

**Consequences:** Security patches in upstream are missed (vulnerability window). New features in upstream require manual backporting. The "standing on the shoulders of giants" advantage evaporates -- you're now maintaining giants. DeerFlow (ByteDance) and Scion (Google Cloud) are actively developed; their velocity will outpace your ability to sync.

**Prevention:**
- Adapter pattern over deep forking: Wrap each system with a thin TypeScript adapter (already planned for DeerFlow/Scion as API clients). Minimize modifications to forked source code. Push customizations into the adapter layer.
- For Ruflo and EverMemOS (consumed directly as TS): isolate modifications in clearly separated files/directories. Never modify upstream files inline -- extend, don't edit.
- Upstream contribution: Where possible, contribute needed changes upstream to reduce divergence.
- Monthly sync cadence: Schedule upstream sync as a recurring task, not a "when we have time" activity. Automate `git fetch upstream` + conflict detection in CI.
- Pin upstream versions: Don't track `main`. Pin to tagged releases and upgrade deliberately.

**Detection:** `git log --oneline upstream/main..HEAD` shows >50 divergent commits. Upstream changelog mentions breaking changes you haven't evaluated. Security advisories for upstream dependencies go unpatched in your fork.

**Phase:** Phase 1 (monorepo setup). Establish the adapter-over-fork pattern and upstream sync process before writing integration code. Changing this later means rewriting all integration points.

**Confidence:** HIGH -- fork drift is extensively documented across the open-source ecosystem and is the #1 reason companies abandon fork strategies.

---

### Pitfall 4: AGPL-3.0 License Contamination (MiroFish)

**What goes wrong:** MiroFish is licensed AGPL-3.0, which is a viral copyleft license. If MiroFish code is linked, imported, or tightly integrated with Ultrabuilder, the AGPL may require the entire product's source code to be released under AGPL-3.0. This is a legal landmine for a commercial product.

**Why it happens:** Developers integrate first and check licenses later. The boundary between "using a library" and "incorporating AGPL code" is legally ambiguous. Google, Apple, and most enterprises ban AGPL internally because the risk is too high relative to the compliance cost.

**Consequences:** If MiroFish is tightly coupled, Ultrabuilder may be legally required to open-source its entire codebase under AGPL. Alternatively, expensive legal review and possible architectural rework to decouple. Investors and enterprise customers will flag AGPL as a dealbreaker during due diligence.

**Prevention:**
- MiroFish is already deferred to Phase 02 -- good. When it is integrated, it MUST run as a completely separate process (network-isolated sidecar) with communication only via a well-defined API (HTTP/gRPC). No shared libraries, no code linking, no importing MiroFish modules into the main codebase.
- Get legal counsel before Phase 02 integration begins. The AGPL "network use" clause is ambiguous for desktop apps.
- Evaluate whether MiroFish's swarm simulation can be replicated with a permissively-licensed alternative, avoiding AGPL entirely.
- Document the license of every dependency in every fork. Transitive AGPL dependencies in DeerFlow/Scion/EverMemOS are equally dangerous.

**Detection:** `license-checker` or `pip-licenses` audit reveals AGPL dependencies in the dependency tree. Code review shows direct imports from MiroFish modules rather than API calls.

**Phase:** Phase 02 (MiroFish integration). But license audit of ALL 5 forks should happen in Phase 1 to catch transitive AGPL/GPL contamination early.

**Confidence:** HIGH -- AGPL implications are well-established in legal precedent and industry practice.

---

### Pitfall 5: Sidecar Lifecycle Hell (Python + Go Services on User Machines)

**What goes wrong:** DeerFlow (Python) and Scion (Go) run as sidecar processes managed by the Tauri desktop app. On user machines, these sidecars crash silently, fail to start due to missing dependencies (Python version mismatch, Docker not installed), leak as orphan processes on unclean shutdown, and behave differently across OS process management models (Windows signals vs Unix signals).

**Why it happens:** Tauri's sidecar support requires significant boilerplate for production-ready lifecycle management. There is no standard health check implementation -- each app must build custom monitoring. Platform differences in process signaling (SIGTERM on Unix vs TerminateProcess on Windows) create race conditions in shutdown handlers. Users' machines are not developer machines -- Python 3.11 vs 3.12, missing pip packages, Docker Desktop not running.

**Consequences:** Users report "app doesn't work" but the Tauri app is fine -- it's the invisible sidecar that died. Orphan Python/Go processes consume memory indefinitely. On Windows, sidecar processes survive app termination and must be manually killed. On macOS, Gatekeeper may block unsigned sidecar binaries. Debugging requires users to check multiple process logs across multiple languages.

**Prevention:**
- Bundle sidecar binaries as single-file executables: Use PyInstaller/PyOxidizer for DeerFlow, `go build` (already static) for Scion. Do not rely on the user's Python installation.
- Implement a process supervisor in the Rust backend: Health check pings every 5s, automatic restart on crash, graceful shutdown with escalating signals (SIGTERM -> wait 5s -> SIGKILL), PID file tracking for orphan cleanup on app start.
- Single log aggregation point: All sidecar logs route to Tauri's log directory with structured JSON. Surface sidecar health in the UI (green/yellow/red indicators).
- Docker dependency check on first launch: If Docker Desktop is not running, show a clear error with instructions, not a cryptic sidecar crash.

**Detection:** Windows Task Manager shows orphaned `python.exe` or `scion` processes after app close. App hangs on shutdown (waiting for sidecar that won't die). Users on fresh OS installs get "connection refused" errors with no explanation.

**Phase:** Phase 1 (sidecar infrastructure). The process supervisor must be built before any agent execution, because every agent depends on DeerFlow/Scion being alive.

**Confidence:** HIGH -- Tauri GitHub issues (#3062, #8821) and community discussions extensively document sidecar lifecycle problems.

---

## Moderate Pitfalls

### Pitfall 6: Git Worktree Explosion and Merge Debt

**What goes wrong:** Each of 12 agents gets a git worktree. Agents modify shared files (package.json, route definitions, schema files, tsconfig). At merge time, conflicts are guaranteed on these "hotspot" files. Worktrees accumulate and are not cleaned up, consuming disk space. Branch proliferation makes the repo history unreadable.

**Prevention:**
- Designate hotspot files as single-writer: One agent owns `package.json`, `tsconfig.json`, route definitions. Others submit changes through the owner via a queue.
- Use additive-only patterns: Agents add new files rather than modifying existing ones. New routes go in separate files that are auto-imported via glob patterns.
- Worktree cleanup on task completion: Automated `git worktree remove` + branch deletion after merge. CI job to detect orphaned worktrees.
- Cap concurrent worktrees: 3-5 active worktrees max based on task dependencies, not 12 simultaneous.
- Run `git merge-tree` (dry merge) between active worktrees periodically to detect conflicts before they accumulate.

**Detection:** `git worktree list` shows >10 worktrees. Merge PRs require manual conflict resolution on >3 files. Disk usage in the repo directory exceeds 5GB.

**Phase:** Phase 1 (Git integration design). The worktree strategy must be defined alongside the agent coordination protocol.

**Confidence:** HIGH -- real-world reports from AI coding agent teams confirm hotspot file conflicts are the primary pain point.

---

### Pitfall 7: SQLite + Postgres Dual-Database Schema Drift

**What goes wrong:** The local SQLite database and remote Postgres database schemas diverge over time. Migrations are applied to one but not the other. Data types that work in Postgres (arrays, JSONB, enums) don't exist in SQLite. Sync logic assumes schema parity that doesn't exist.

**Prevention:**
- Use a sync framework (PowerSync, ElectricSQL, or sqlite-sync) rather than building custom sync. These handle schema translation and conflict resolution.
- Shared schema definition: One source of truth (TypeScript types + Drizzle/Prisma schema) that generates both SQLite and Postgres migrations.
- Use UUID/ULID primary keys everywhere, never auto-incrementing integers. Auto-increment causes ID collision on sync.
- Limit Postgres-specific features: No arrays, no JSONB, no enums in synced tables. Use JSON text columns and string enums that work in both.
- Schema migration CI: Every migration must pass against both SQLite and Postgres in CI.

**Detection:** Sync failures with "column not found" or "type mismatch" errors. Data present in local SQLite but missing from Postgres (or vice versa). Migration count differs between databases.

**Phase:** Phase 1 (database setup). Schema strategy must be decided before any data models are created. Changing primary key strategy later requires a data migration.

**Confidence:** MEDIUM -- well-known pattern in local-first apps, but specific tooling (PowerSync, ElectricSQL) has matured significantly and reduces risk if adopted early.

---

### Pitfall 8: WebView Memory Pressure from Real-Time Agent Streaming

**What goes wrong:** Streaming output from 12 agents simultaneously means 12 concurrent SSE/WebSocket connections, each pushing text updates multiple times per second. The WebView's DOM grows unboundedly as agent output accumulates. After 30-60 minutes of a build session, the UI becomes unresponsive, memory exceeds 2-4GB, and the app may crash.

**Prevention:**
- Virtualized rendering: Only render visible agent output. Use virtual scrolling (react-window/react-virtuoso) for agent log panels. Off-screen agents' output goes to a ring buffer, not the DOM.
- Connection pooling: Multiplex all agent streams over a single WebSocket connection with message framing, not 12 separate connections.
- Output throttling: Batch DOM updates at 100ms intervals, not on every token. Users can't read faster than 100ms updates anyway.
- Memory ceiling: Ring buffer per agent (last 1000 lines in DOM, full log in Rust backend). Old output is evicted from the WebView and retrievable on scroll-back from the backend.
- Monitor `performance.memory` (Chrome/WebView2) and `window.performance` metrics. Alert when heap exceeds threshold.

**Detection:** App memory usage grows linearly over time during builds. UI frame rate drops below 30fps during active streaming. Agent panels lag behind actual execution by >5 seconds.

**Phase:** Phase 1 (Build & Test Studio UI). Virtualized rendering must be the default from the start. Retrofitting virtualization onto a naive "append to div" implementation is a significant rewrite.

**Confidence:** HIGH -- WebView memory issues with streaming content are extensively documented for Electron/Tauri apps.

---

### Pitfall 9: Docker Desktop Dependency on User Machines

**What goes wrong:** DeerFlow agents execute inside Docker containers. Docker Desktop is a separate commercial product that users must install, keep running, and (on macOS/Windows) allocate sufficient resources to. Many users don't have Docker installed. Those who do may have resource limits (2GB RAM default on Docker Desktop) that are insufficient for concurrent agent containers.

**Prevention:**
- First-launch onboarding: Detect Docker presence and version. If missing, show clear instructions. If Docker daemon is not running, prompt to start it.
- Resource requirement check: Query Docker's resource limits (`docker info`) and warn if RAM < 4GB or disk < 20GB free.
- Fallback execution mode: For simple tasks, offer a non-Docker execution path (direct process execution) as degraded mode. Not all agent tasks require container isolation.
- Container resource limits: Set explicit `--memory` and `--cpus` flags on agent containers to prevent one agent from consuming all Docker resources.
- Document system requirements prominently: "Requires Docker Desktop with 8GB RAM allocated" in installation docs.

**Detection:** Agent execution hangs with no output (Docker not running). Container OOM kills mid-build. "Cannot connect to Docker daemon" errors in sidecar logs.

**Phase:** Phase 1 (DeerFlow integration). The Docker dependency is a hard prerequisite for the core pipeline. Must be addressed in onboarding flow.

**Confidence:** HIGH -- Docker Desktop dependency is a well-known friction point for developer tools.

---

## Minor Pitfalls

### Pitfall 10: Tauri Auto-Update Key Loss

**What goes wrong:** Tauri's updater requires a cryptographic key pair to sign updates. If the private key is lost, existing users can never receive updates -- they must manually reinstall. This is not recoverable.

**Prevention:** Store the signing key in a secrets manager (1Password, AWS Secrets Manager, not a local file). Back up the key in at least two locations. Document the key location in team runbook. Generate the key in CI, not on a developer's laptop.

**Phase:** Phase 1 (distribution setup). Generate and secure the key before the first release.

**Confidence:** HIGH -- stated explicitly in Tauri documentation.

---

### Pitfall 11: Windows SmartScreen Blocking Unsigned Binaries

**What goes wrong:** Self-signed or newly-signed Windows binaries trigger SmartScreen warnings ("Windows protected your PC"). Users see a scary warning and abandon installation. Building reputation with SmartScreen requires an EV (Extended Validation) certificate, which costs $300-500/year and requires hardware token delivery.

**Prevention:** Budget for an EV code signing certificate before the first Windows release. Factor in 2-4 week procurement time. Use Azure Trusted Signing or a traditional CA (DigiCert, Sectigo). Self-signed certificates are acceptable only for internal/beta testing.

**Phase:** Pre-release (before first public Windows build).

**Confidence:** HIGH -- standard Windows distribution requirement.

---

### Pitfall 12: Polyglot Observability Blind Spots

**What goes wrong:** A user reports "the build failed." The failure could be in: the TypeScript UI, the Rust Tauri backend, the Python DeerFlow sidecar, the Go Scion sidecar, or inside a Docker container managed by DeerFlow. Each component logs in a different format, to a different location, with no trace correlation. Debugging requires manually correlating timestamps across 4+ log sources.

**Prevention:**
- Unified trace ID: Generate a request ID in the Tauri backend and propagate it through all sidecar API calls and into Docker container environments.
- Structured JSON logging everywhere: All components emit `{timestamp, trace_id, level, component, message}`.
- Single log aggregation: Sidecars write to Tauri's log directory or stream logs via API. The UI surfaces a unified timeline.
- Error classification: Each component reports errors in a standard envelope so the UI can show "DeerFlow container crashed" not "unknown error."

**Phase:** Phase 1 (sidecar infrastructure). Observability plumbing is cheap to add early and extremely expensive to retrofit.

**Confidence:** MEDIUM -- standard polyglot systems challenge, well-understood mitigation patterns.

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Phase 1: Tauri shell | WebView divergence (Pitfall 1), key loss (Pitfall 10) | Cross-platform CI from day 1, secure signing key immediately |
| Phase 1: Flow Engine | Agent coordination collapse (Pitfall 2) | Design coordination protocol before agent implementation |
| Phase 1: Monorepo setup | Fork drift starts here (Pitfall 3) | Adapter pattern, minimize upstream modifications |
| Phase 1: Sidecar infra | Lifecycle hell (Pitfall 5), Docker dep (Pitfall 9) | Process supervisor, bundled binaries, onboarding checks |
| Phase 1: Database | Schema drift (Pitfall 7) | Choose sync framework, UUID keys, shared schema source |
| Phase 1: Git integration | Worktree explosion (Pitfall 6) | Single-writer rule, additive patterns, worktree cap |
| Phase 1: Build Studio UI | Memory pressure (Pitfall 8) | Virtualized rendering from the start |
| Phase 1: Observability | Blind spots (Pitfall 12) | Unified trace ID, structured logging, log aggregation |
| Phase 02: MiroFish | AGPL contamination (Pitfall 4) | Process isolation only, legal review before integration |
| Pre-release: Distribution | SmartScreen (Pitfall 11) | EV certificate procurement 4 weeks before release |

---

## Sources

- [Tauri WebView Versions documentation](https://v2.tauri.app/reference/webview-versions/)
- [WebKit GTK instability discussion](https://github.com/orgs/tauri-apps/discussions/8524)
- [Tauri sidecar lifecycle management issue #3062](https://github.com/tauri-apps/plugins-workspace/issues/3062)
- [Tauri optional sidecars issue #8821](https://github.com/tauri-apps/tauri/issues/8821)
- [Evil Martians: Tauri + Rust sidecar guide](https://evilmartians.com/chronicles/making-desktop-apps-with-revved-up-potential-rust-tauri-sidecar)
- [Multi-agent 17x error trap analysis (Towards Data Science)](https://towardsdatascience.com/why-your-multi-agent-system-is-failing-escaping-the-17x-error-trap-of-the-bag-of-agents/)
- [Multi-agent coordination failures (Galileo)](https://galileo.ai/blog/multi-agent-ai-failures-prevention)
- [Git worktree conflicts with AI agents (Termdock)](https://www.termdock.com/en/blog/git-worktree-conflicts-ai-agents)
- [Clash: merge conflict detection across worktrees](https://github.com/clash-sh/clash)
- [Git worktrees for multi-agent development (Vibehackers)](https://vibehackers.io/blog/git-worktrees-multi-agent-development)
- [Fork drift dangers (Preset)](https://preset.io/blog/stop-forking-around-the-hidden-dangers-of-fork-drift-in-open-source-adoption/)
- [AGPL license implications (Open Core Ventures)](https://www.opencoreventures.com/blog/agpl-license-is-a-non-starter-for-most-companies)
- [Google AGPL policy](https://opensource.google/documentation/reference/using/agpl-policy)
- [PowerSync: Postgres-SQLite sync](https://www.powersync.com/blog/introducing-powersync-v1-0-postgres-sqlite-sync-layer)
- [ElectricSQL: local-first sync](https://electric-sql.com/blog/2023/09/20/introducing-electricsql-v0.6)
- [WebView2 memory overhead (Microsoft)](https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/performance)
- [Tauri updater documentation](https://v2.tauri.app/plugin/updater/)
- [Tauri code signing guide](https://dev.to/tomtomdu73/ship-your-tauri-v2-app-like-a-pro-code-signing-for-macos-and-windows-part-12-3o9n)
- [Tauri distribution documentation](https://v2.tauri.app/distribute/)
