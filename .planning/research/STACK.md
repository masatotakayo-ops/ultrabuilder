# Technology Stack

**Project:** Ultrabuilder
**Researched:** 2026-03-28
**Overall confidence:** HIGH

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

```bash
# Root monorepo setup
npm create turbo@latest

# Core frontend (in apps/desktop)
npm install react@^19.2 react-dom@^19.2
npm install @tauri-apps/api@^2
npm install @tauri-apps/plugin-sql@^2
npm install @tauri-apps/plugin-shell@^2
npm install @tauri-apps/plugin-http@^2
npm install @tauri-apps/plugin-fs@^2
npm install @tauri-apps/plugin-dialog@^2
npm install @tauri-apps/plugin-process@^2
npm install @tauri-apps/plugin-updater@^2

# UI framework
npm install tailwindcss@^4
npx shadcn@latest init

# State management
npm install zustand@^5
npm install @tanstack/react-query@^5

# IDE components
npm install @monaco-editor/react@next  # React 19 compat
npm install @xyflow/react@^12.10
npm install @xterm/xterm@^6
npm install react-xtermjs

# API sidecar (in packages/api or separate)
npm install hono@^4.12

# Database
npm install drizzle-orm@^0.45
npm install -D drizzle-kit

# Dev dependencies
npm install -D typescript@^6
npm install -D @biomejs/biome@^2
npm install -D vitest@^4
npm install -D @testing-library/react
npm install -D @playwright/test
npm install -D vite@^6
```

```toml
# Cargo.toml (src-tauri) - Rust dependencies
[dependencies]
tauri = { version = "2.10", features = ["all"] }
tauri-plugin-sql = { version = "2.3", features = ["sqlite"] }
tauri-plugin-shell = "2"
tauri-plugin-http = "2"
tauri-plugin-fs = "2"
tauri-plugin-dialog = "2"
tauri-plugin-process = "2"
tauri-plugin-updater = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tokio = { version = "1", features = ["full"] }
bollard = "0.18"  # Docker API
git2 = "0.19"     # Git operations (libgit2 bindings)
```

## Version Pinning Strategy

- **Pin major.minor** for core dependencies (Tauri, React, Tailwind). Breaking changes are rare within patch versions.
- **Use caret (^)** for supporting libraries (Zustand, TanStack Query, Hono). These follow semver well.
- **Lock exact** for Biome and Drizzle during pre-1.0 phases. Both are moving fast.
- **Turborepo lockfile**: Use `packageManager` field in root package.json. Recommend pnpm for monorepo (faster installs, strict hoisting).

## Package Manager Recommendation

Use **pnpm** over npm/yarn:
- Strict dependency resolution (no phantom dependencies)
- Workspace protocol for monorepo packages
- 2-3x faster installs via content-addressable storage
- Native Turborepo support

## Key Integration Notes

### Clerk + Tauri Workaround
Clerk's standard web SDK doesn't work in Tauri's WebView due to cookie handling on custom protocols. Use `tauri-plugin-clerk` which patches `globalThis.fetch` to route through `tauri-plugin-http`. OAuth and magic links do NOT work -- use email/password or custom JWT token exchange. Plan for this constraint.

### Drizzle + Tauri Proxy Pattern
Drizzle's `readMigrationFiles()` uses Node's `fs` module which doesn't exist in WebView. Use the proxy pattern: Drizzle computes SQL in the frontend, sends query + params to a Tauri Rust command that executes via sqlx, returns JSON. Define a custom `run_sql` Tauri command.

### Monaco in Tauri WebView
Monaco's web workers need special handling in Tauri. Use `@monaco-editor/react`'s loader to configure worker paths. The `MonacoEnvironment.getWorkerUrl` must point to correctly bundled worker files. Test early.

### Hono Sidecar Compilation
Compile the Hono server into a single binary using `@yao-pkg/pkg`. Configure Tauri's `externalBin` in `tauri.conf.json` with target triple suffixes. The sidecar starts on app launch, communicates via localhost HTTP. Manage lifecycle via Tauri's shell plugin.

### Docker from Tauri
Use the `bollard` Rust crate in Tauri commands for Docker operations. This avoids spawning shell processes and gives type-safe Docker API access. For complex orchestration (compose-like), shell out to `docker compose` via Tauri's Command API.

### Git from Tauri
Use the `git2` Rust crate (libgit2 bindings) for all git operations in Tauri commands: worktree creation/removal, branch management, commit orchestration. This is type-safe, avoids shell-out overhead, and has no dependency on git being installed in the user's PATH. The git2 crate supports the full worktree API needed for agent isolation.

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
