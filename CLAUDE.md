<!-- GSD:project-start source:PROJECT.md -->
## Project

**Ultrabuilder**

Ultrabuilder is an autonomous software factory — a Tauri desktop application that turns product intent into production-ready commercial SaaS through wizard-driven Connected Studios, a coordinated agent swarm, and deep GitHub integration. Users define what they want to build, and the platform's Flow Engine (Ruflo + DeerFlow + Scion + EverMemOS) plans, generates, tests, and deploys the software into a GitHub repository. Built for developers and teams who want AI-powered software synthesis with full traceability.

**Core Value:** The core pipeline must work: a user describes what they want → the system plans it → agents build it into a GitHub repo with real, working code. Everything else is secondary to this golden path.

### Constraints

- **Tech stack**: Tauri 2.0 (Rust + WebView), React + TypeScript, shadcn/ui + Tailwind, Turborepo monorepo — chosen for IDE-like requirements (file system, Docker, git, long-running agents)
- **Auth**: Clerk with Organizations from day one — single-user now, team-ready architecture
- **Database**: SQLite local + Postgres for team/cloud sync — local-first, sync when connected
- **Agent execution**: DeerFlow runs in Docker containers locally — no serverless timeout limits
- **Git**: Every agent action must commit to the connected GitHub repo with structured messages
- **Security**: Ruflo's 7-layer Guidance Control Plane enforces policy gates agents cannot bypass
- **Desktop distribution**: Tauri auto-updater for releases, cross-platform (macOS, Windows, Linux)
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Core Shell
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Tauri | 2.10.x | Desktop shell (Rust + WebView) | Native file system, Docker CLI, git access. Only viable option for an IDE-like desktop app that needs OS-level access without Electron's memory overhead. Stable since Oct 2024, actively maintained. | HIGH |
| TypeScript | 6.0.x | Language | Latest stable (March 2026). Last JS-based release before TS 7 (Go rewrite). Use strict mode. | HIGH |
| Rust (stable) | latest | Tauri backend, Tauri commands, plugin authoring | Required by Tauri. Use for performance-critical paths: file watching, git operations, Docker API calls. | HIGH |
### Frontend Framework
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| React | 19.2.x | UI framework | Stable, mature ecosystem. Required by Monaco, React Flow, and xterm wrappers. v19.2 has improved async orchestration and Suspense. | HIGH |
| Tailwind CSS | 4.x | Utility-first styling | v4 is CSS-first config (@theme), 5x faster builds, cascade layers. shadcn/ui v4 is built for it. | HIGH |
| shadcn/ui | CLI v4 | Component library | Not a dependency -- copies components into your project. CLI v4 (March 2026) supports Base UI primitives, RTL, app blocks. Perfect for IDE chrome: panels, dialogs, command palette, sidebars. | HIGH |
### State Management
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Zustand | 5.x | Client-side state | ~1KB, no providers, excellent TypeScript support. Ideal for complex IDE state (panel layout, active files, agent status). Slices pattern scales well. | HIGH |
| TanStack Query | 5.x | Server/async state | Handles caching, background refetch, optimistic updates for API calls to Hono sidecar, Flow Engine services, and GitHub API. Pairs naturally with Zustand. | HIGH |
### IDE Components
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| @monaco-editor/react | 4.7.x | Code editor | VS Code's editor. The only serious option for syntax highlighting, intellisense, diffing. Use v4.7.0-rc.0 for React 19 compat. | HIGH |
| @xyflow/react | 12.10.x | Architecture canvas | Node-based UI for agent workflow visualization, architecture diagrams. React Flow UI components support shadcn CLI and Tailwind CSS 4. Actively maintained by xyflow team. | HIGH |
| @xterm/xterm | 6.x | Terminal emulator | v6 (Dec 2025) is latest. WebGL renderer, ligatures, progress addon. For agent output streams and build logs. | HIGH |
| react-xtermjs | latest | React wrapper for xterm | Qovery-maintained, supports hooks and addons. Only actively maintained React wrapper for xterm.js. | MEDIUM |
### Backend / API
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Hono | 4.12.x | Local API server (sidecar) | Ultrafast, Web Standards based, ~14KB. Runs as Node.js sidecar binary compiled with @yao-pkg/pkg. Handles auth middleware, Flow Engine routing, GitHub proxy. | HIGH |
| Drizzle ORM | 0.45.x (v1-beta available) | Database ORM | Type-safe SQL, works with both SQLite and Postgres. Use proxy mode for Tauri (frontend computes SQL, backend executes via sqlx). Zero runtime overhead. | HIGH |
### Database
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| tauri-plugin-sql (SQLite) | 2.3.x | Local database | Official Tauri plugin, uses sqlx under the hood. Migrations via Drizzle in proxy pattern. Stores projects, agent state, local config, memory cache. | HIGH |
| PostgreSQL | 16+ | Cloud sync / team database | For future team features. Drizzle supports both SQLite and Postgres with same schema definitions. Not needed Phase 1. | MEDIUM |
### Authentication
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Clerk | latest | Auth + user management | Organizations model matches role system. **CAVEAT:** Tauri integration is community-maintained (tauri-plugin-clerk). OAuth flows are broken in WebView -- must use email/password or custom token flow. Requires patching global fetch for Tauri's HTTP plugin. | MEDIUM |
### Monorepo / Build
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Turborepo | 2.8.x | Monorepo orchestration | Incremental builds, task graph, remote caching. Written in Rust. Devtools (2.7+) for visualizing package/task graphs. Ideal for the packages/* structure. | HIGH |
| Vite | 6.x | Frontend bundler | Fast HMR, native ESM. Tauri's default frontend bundler. Powers Vitest. | HIGH |
| Biome | 2.x | Linting + formatting | Single binary, 25x faster than ESLint+Prettier. 423+ lint rules, type-aware linting. For a monorepo with many packages, build speed matters. | HIGH |
### Testing
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Vitest | 4.x | Unit + integration tests | Native ESM/TS/JSX, Vite-powered, Jest-compatible API. v4 has stable browser mode. | HIGH |
| Playwright | latest | E2E tests | Tauri supports WebDriver testing. Playwright for cross-platform E2E of the desktop app. | HIGH |
| React Testing Library | latest | Component tests | Standard for React component testing with Vitest. | HIGH |
### Infrastructure / DevOps
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Docker Engine API | v1.47+ | Agent sandbox orchestration | Call Docker API from Rust (bollard crate). Manages containers for DeerFlow/Scion agent sandboxes. Type-safe, async, no shell dependency. | HIGH |
| git2 (Rust crate) | 0.19.x | Git worktree management | Rust-native libgit2 bindings for worktree create/list/remove from Tauri commands. Each agent gets an isolated worktree. Type-safe, no PATH dependency on git CLI. | HIGH |
| @yao-pkg/pkg | latest | Node.js binary compilation | Compiles Hono sidecar into single binary for Tauri sidecar embedding. Community fork of vercel/pkg, actively maintained. | MEDIUM |
## Alternatives Considered
| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| Desktop shell | Tauri 2 | Electron | 3-5x memory overhead, ships Chromium. Tauri is the right call for an IDE. |
| Desktop shell | Tauri 2 | Neutralinojs | Smaller ecosystem, fewer plugins, less mature. |
| State mgmt | Zustand | Redux Toolkit | Boilerplate-heavy for the number of stores an IDE needs. Zustand is simpler with same capability. |
| State mgmt | Zustand | Jotai | Atomic model is better for forms, not complex interconnected IDE state. |
| ORM | Drizzle | Prisma | Prisma requires a binary engine, doesn't work in WebView, heavier. Drizzle's proxy mode is purpose-built for Tauri. |
| ORM | Drizzle | TypeORM | Legacy patterns, decorator-based, poor TypeScript inference. |
| Linting | Biome | ESLint + Prettier | 6 packages to configure vs 1 binary. In a monorepo, Biome's speed advantage is material. |
| Testing | Vitest | Jest | Vitest is Vite-native, faster, same API. No reason to use Jest in a Vite project. |
| API framework | Hono | Express | Express is heavier, not Web Standards based, slower. Hono is purpose-built for edge/lightweight contexts. |
| API framework | Hono | Fastify | Fastify is Node-only, heavier. Hono works on Bun/Deno/Node, tiny footprint for a sidecar. |
| Components | shadcn/ui | Radix + custom | shadcn/ui IS Radix underneath but with pre-styled Tailwind components. No reason to go lower-level. |
| Components | shadcn/ui | Ant Design / MUI | CSS-in-JS runtime overhead, harder to customize, foreign to Tailwind ecosystem. |
| Terminal | xterm.js | custom terminal | xterm.js is what VS Code uses. No realistic alternative. |
| Editor | Monaco | CodeMirror 6 | Monaco has richer API for IDE features (intellisense, diffing, diagnostics). CodeMirror is lighter but less capable. |
| Auth | Clerk | Auth.js/NextAuth | NextAuth is web-framework-specific. Clerk works (with workarounds) in desktop contexts and has Organizations. |
| Auth | Clerk | Supabase Auth | Supabase Auth lacks the Organizations/roles model needed for the PRD's role system. |
| Monorepo | Turborepo | Nx | Nx is more opinionated, heavier, plugin-centric. Turborepo is simpler, faster for JS/TS monorepos. |
| Bundler | Vite | Webpack | Vite is Tauri's default, faster, simpler config. Webpack is legacy for new projects. |
| Docker API | bollard (Rust) | dockerode (Node) | Prefer Rust-side Docker calls via Tauri commands for performance. Fall back to dockerode in Hono sidecar if needed. |
| Git operations | git2 (Rust) | simple-git (Node) | Rust-native avoids shell-out overhead, works in Tauri commands directly, no git CLI PATH dependency. |
## Installation
# Root monorepo setup
# Core frontend (in apps/desktop)
# UI framework
# State management
# IDE components
# API sidecar (in packages/api or separate)
# Database
# Dev dependencies
# Cargo.toml (src-tauri) - Rust dependencies
## Version Pinning Strategy
- **Pin major.minor** for core dependencies (Tauri, React, Tailwind). Breaking changes are rare within patch versions.
- **Use caret (^)** for supporting libraries (Zustand, TanStack Query, Hono). These follow semver well.
- **Lock exact** for Biome and Drizzle during pre-1.0 phases. Both are moving fast.
- **Turborepo lockfile**: Use `packageManager` field in root package.json. Recommend pnpm for monorepo (faster installs, strict hoisting).
## Package Manager Recommendation
- Strict dependency resolution (no phantom dependencies)
- Workspace protocol for monorepo packages
- 2-3x faster installs via content-addressable storage
- Native Turborepo support
## Key Integration Notes
### Clerk + Tauri Workaround
### Drizzle + Tauri Proxy Pattern
### Monaco in Tauri WebView
### Hono Sidecar Compilation
### Docker from Tauri
### Git from Tauri
## Sources
- Tauri 2.10.3: https://docs.rs/crate/tauri/latest, https://v2.tauri.app/
- TypeScript 6.0: https://devblogs.microsoft.com/typescript/announcing-typescript-6-0/
- React 19.2: https://react.dev/versions
- Tailwind CSS 4: https://tailwindcss.com/blog/tailwindcss-v4
- shadcn/ui CLI v4: https://ui.shadcn.com/docs/changelog/2026-03-cli-v4
- Zustand 5: https://github.com/pmndrs/zustand
- TanStack Query 5: https://tanstack.com/query/latest
- Monaco Editor React: https://www.npmjs.com/package/@monaco-editor/react
- React Flow 12: https://reactflow.dev/, https://www.npmjs.com/package/@xyflow/react
- xterm.js 6: https://xtermjs.org/, https://github.com/xtermjs/xterm.js/releases
- Hono 4.12: https://hono.dev/
- Drizzle ORM: https://orm.drizzle.team/
- Drizzle + Tauri proxy pattern: https://dev.to/huakun/drizzle-sqlite-in-tauri-app-kif
- Turborepo 2.8: https://turborepo.dev/
- Biome 2: https://github.com/biomejs/biome
- Vitest 4: https://vitest.dev/
- Clerk + Tauri: https://github.com/Nipsuli/tauri-plugin-clerk
- tauri-plugin-sql: https://v2.tauri.app/plugin/sql/
- bollard (Docker): https://docs.rs/bollard/latest/bollard/
- git2 (libgit2): https://docs.rs/git2/latest/git2/
- @yao-pkg/pkg: https://github.com/yao-pkg/pkg
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
