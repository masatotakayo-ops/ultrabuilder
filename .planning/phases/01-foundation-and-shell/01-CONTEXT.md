# Phase 1: Foundation and Shell - Context

**Gathered:** 2026-03-28
**Status:** Ready for planning

<domain>
## Phase Boundary

Deliver a working Tauri desktop application that launches cross-platform (macOS, Windows, Linux) with a React UI, SQLite local persistence, a monorepo build pipeline, a proven IPC layer (Commands + Channels + Events), and a cross-platform CI pipeline. No Studios, no sidecars, no agent execution. This is the permanent skeleton all future phases slot into.

</domain>

<decisions>
## Implementation Decisions

### App Shell Layout
- **D-01:** VS Code-style layout — activity bar (far left) → collapsible sidebar → main editor area → bottom panel. This is the permanent skeleton; Phase 5 Studios slot into main area and sidebar.
- **D-02:** Sidebar is collapsible from day one via a toggle button. Pattern is cheap to establish now, expensive to retrofit later.
- **D-03:** Phase 1 content in the main area is a Projects screen — list of local projects with name/date, create + open actions. Maps directly to the SQLite schema.
- **D-04:** TanStack Router is set up in Phase 1. Future phases add routes cleanly (e.g., `/projects/:id/studio/prd`).

### SQLite Schema
- **D-05:** Projects table only in Phase 1. No stub tables — schema for Phase 5 entities (agent_runs, secrets, build_plans) added via migration when those phases execute.
- **D-06:** Projects fields: `id` (uuid), `name`, `description`, `github_repo` (nullable — wired in Phase 5), `status` (enum: planning|building|complete), `created_at`, `updated_at`.
- **D-07:** Drizzle migrations from day one. Migration files committed to repo. `drizzle-kit generate` + `migrate` on app startup.
- **D-08:** Drizzle schema types (`schema.ts`, `types.ts`) live in `packages/shared`. DB → API → UI share one type definition.

### IPC Proof Points
- **D-09:** Commands demonstrated via database CRUD — `create_project`, `list_projects`, `update_project`, `delete_project`. These are the real Commands Phase 5 uses directly; no throwaway demo code.
- **D-10:** Channels demonstrated via log streaming — Rust-side `tracing` events stream to a bottom-panel console. Exact same pattern Phase 3 uses for agent output.
- **D-11:** Events demonstrated via app lifecycle — `app:ready`, `window:focus`, `window:blur` emitted by Rust, consumed by React global state. Phase 2 extends with `sidecar:started`, `sidecar:crashed`.
- **D-12:** Dev-only IPC debug panel in the bottom bar (visible when `NODE_ENV=development`). Shows live IPC events with type, payload, and timestamp. Hidden in production builds.

### Window Chrome & Platform Feel
- **D-13:** Custom titlebar — draggable, shows app name, VS Code style. Consistent across macOS/Windows/Linux via Tauri `decorations: false` + custom CSS drag region.
- **D-14:** Full IDE native app menu from day one: File (New Project, Open Project, Quit), Edit (Undo, Redo, Cut, Copy, Paste), View (Toggle Sidebar, Toggle Bottom Panel, Zoom), Window (Minimize, Zoom, Close), Help (About). Tauri `Menu` API (Rust side).
- **D-15:** System-aware theme — respects OS light/dark preference via `prefers-color-scheme` media query + shadcn dark mode class toggle. Both themes tested in CI.
- **D-16:** Geist Sans for all UI text, Geist Mono for code, paths, IDs, timestamps, and the IPC debug panel. Fonts bundled in app (no CDN dependency).

### Claude's Discretion
- Activity bar icons and ordering (can use shadcn icons or lucide-react)
- Exact bottom panel tab layout (Log | IPC Debug | Problems)
- Loading/empty states on Projects screen
- Window minimum size constraints
- Rust module structure inside src-tauri/

</decisions>

<specifics>
## Specific Ideas

- The IPC debug panel should feel like Chrome DevTools network tab — event type, direction (→ Rust / ← React), payload, timestamp
- The Projects screen is the Phase 1 "home" — no splash screen, land directly on projects list
- Full IDE menu matters: macOS users expect a real menu bar; shipping without one feels broken

</specifics>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project constraints and architecture
- `.planning/PROJECT.md` — Constraints section (tech stack, auth, database choices that are locked)
- `.planning/PROJECT.md` — Architecture section (monorepo structure: apps/desktop, packages/*)
- `.planning/REQUIREMENTS.md` §Foundation — FNDN-01 through FNDN-05 (exact requirements for this phase)

### Technology decisions (from research)
- `.planning/research/STACK.md` — Confirmed versions: Tauri 2.10, React 19.2, TS 6.0, Tailwind 4, shadcn/ui v4, TanStack Router 5, Zustand 5, Drizzle 0.45, Biome 2, Vitest 4
- `.planning/research/ARCHITECTURE.md` — Three-tier IPC spec, hub-and-spoke model, Drizzle proxy pattern details
- `.planning/research/PITFALLS.md` — Clerk OAuth broken in WebView (Phase 6 concern), WebView memory pressure patterns to avoid in Phase 1

### Phase 1 success criteria
- `.planning/ROADMAP.md` §Phase 1 — 5 success criteria that must all be TRUE for phase completion

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- None — clean repo. Phase 1 establishes all patterns.

### Established Patterns
- None yet — Phase 1 is the pattern-setter. Key patterns to establish:
  - Drizzle proxy pattern (frontend computes SQL → Rust executes via sqlx)
  - Tauri Command handler structure (`#[tauri::command]` + `invoke_handler`)
  - React component co-location (component + types + styles in same folder)
  - Zustand slice pattern for global state (one slice per domain: projects, ipc, ui)

### Integration Points
- Phase 2 adds `sidecar:started` / `sidecar:crashed` Events to the event system established here
- Phase 3 uses the Channel streaming pattern established by log streaming
- Phase 5 adds routes to TanStack Router and adds Studio UI into the VS Code shell established here
- Phase 6 wraps the app in Clerk auth — the app shell needs a protected route wrapper ready

</code_context>

<deferred>
## Deferred Ideas

- Full file explorer in sidebar — Phase 5 (Build & Test Studio)
- Agent live feed panel — Phase 5
- Monaco editor in main area — Phase 5
- Integrated terminal — Phase 5
- Onboarding wizard / Docker detection — Phase 2
- Clerk authentication gates — Phase 6
- System tray icon — post-Phase 1 if needed

</deferred>

---

*Phase: 01-foundation-and-shell*
*Context gathered: 2026-03-28*
