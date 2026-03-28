# Phase 1: Foundation and Shell - Research

**Researched:** 2026-03-28
**Domain:** Tauri 2.0 desktop app scaffolding, monorepo setup, IPC layer, SQLite persistence, cross-platform CI
**Confidence:** HIGH

## Summary

Phase 1 delivers the permanent skeleton for Ultrabuilder: a Tauri 2.0 desktop app with React 19.2 UI, SQLite persistence via Drizzle ORM proxy pattern, a proven IPC layer (Commands + Channels + Events), a Turborepo monorepo with pnpm workspaces, and a cross-platform CI pipeline. No sidecars, no agents, no Studios -- just the foundation all future phases slot into.

The technical approach is well-established. Tauri 2.0 has been stable since October 2024 and is at version 2.10.3. The Drizzle sqlite-proxy pattern for Tauri is documented with working reference implementations. TanStack Router file-based routing with Vite is the standard approach. The primary risks are WebView divergence across platforms (Linux WebKit GTK) and getting the custom window chrome right on all three OSes.

**Primary recommendation:** Scaffold with `create-tauri-app` (React + TypeScript + Vite), restructure into the Turborepo monorepo layout, then implement the IPC proof points (project CRUD, log streaming, lifecycle events) as the real patterns all future phases depend on.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- **D-01:** VS Code-style layout -- activity bar (far left) -> collapsible sidebar -> main editor area -> bottom panel. This is the permanent skeleton; Phase 5 Studios slot into main area and sidebar.
- **D-02:** Sidebar is collapsible from day one via a toggle button.
- **D-03:** Phase 1 content in the main area is a Projects screen -- list of local projects with name/date, create + open actions.
- **D-04:** TanStack Router is set up in Phase 1. Future phases add routes cleanly.
- **D-05:** Projects table only in Phase 1. No stub tables.
- **D-06:** Projects fields: `id` (uuid), `name`, `description`, `github_repo` (nullable), `status` (enum: planning|building|complete), `created_at`, `updated_at`.
- **D-07:** Drizzle migrations from day one. Migration files committed to repo. `drizzle-kit generate` + `migrate` on app startup.
- **D-08:** Drizzle schema types live in `packages/shared`.
- **D-09:** Commands demonstrated via database CRUD -- `create_project`, `list_projects`, `update_project`, `delete_project`.
- **D-10:** Channels demonstrated via log streaming -- Rust `tracing` events stream to bottom-panel console.
- **D-11:** Events demonstrated via app lifecycle -- `app:ready`, `window:focus`, `window:blur`.
- **D-12:** Dev-only IPC debug panel in bottom bar (visible when `NODE_ENV=development`).
- **D-13:** Custom titlebar -- draggable, VS Code style. `decorations: false` + CSS drag region.
- **D-14:** Full IDE native app menu: File, Edit, View, Window, Help. Tauri Menu API (Rust side).
- **D-15:** System-aware theme -- `prefers-color-scheme` + shadcn dark mode class toggle.
- **D-16:** Geist Sans + Geist Mono fonts bundled in app.

### Claude's Discretion
- Activity bar icons and ordering (use lucide-react)
- Exact bottom panel tab layout (Log | IPC Debug | Problems)
- Loading/empty states on Projects screen
- Window minimum size constraints
- Rust module structure inside src-tauri/

### Deferred Ideas (OUT OF SCOPE)
- Full file explorer in sidebar -- Phase 5
- Agent live feed panel -- Phase 5
- Monaco editor in main area -- Phase 5
- Integrated terminal -- Phase 5
- Onboarding wizard / Docker detection -- Phase 2
- Clerk authentication gates -- Phase 6
- System tray icon -- post-Phase 1
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| FNDN-01 | Tauri 2.0 desktop app launches on macOS, Windows, and Linux with React UI | Tauri 2.10.3 scaffolding via create-tauri-app, custom titlebar, platform-specific window chrome, Geist fonts, shadcn/ui + Tailwind 4 |
| FNDN-02 | Turborepo monorepo with apps/desktop and packages/ structure builds successfully | pnpm workspaces, turbo.json task graph, packages/shared for Drizzle types, single `pnpm turbo build` command |
| FNDN-03 | SQLite database persists project state locally via Drizzle ORM proxy pattern | Drizzle sqlite-proxy driver, tauri-plugin-sql, Rust run_sql command, migration-on-startup, projects CRUD |
| FNDN-04 | Cross-platform CI pipeline runs on all 3 OS targets | GitHub Actions matrix with tauri-apps/tauri-action, macOS ARM+Intel, Ubuntu 22.04, Windows latest |
| FNDN-05 | Tauri IPC layer supports Commands, Channels, and Events | Project CRUD commands, log streaming Channel, app lifecycle Events, IPC debug panel |
</phase_requirements>

## Standard Stack

### Core (Phase 1 only -- subset of full stack)

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Tauri | 2.10.3 | Desktop shell (Rust + WebView) | Stable since Oct 2024. Only viable option for IDE-like desktop. |
| @tauri-apps/api | 2.10.1 | Frontend Tauri bindings | Official JS API for invoke, Channel, events |
| @tauri-apps/cli | 2.10.1 | Tauri CLI (dev/build) | Official build tooling |
| @tauri-apps/plugin-sql | 2.3.2 | SQLite via sqlx | Official plugin, wraps sqlx |
| @tauri-apps/plugin-process | 2.3.1 | App lifecycle (exit, relaunch) | Needed for menu Quit action |
| React | 19.2.4 | UI framework | Mature, required by downstream IDE components |
| TypeScript | 6.0.2 | Language | Latest stable, strict mode |
| Vite | 8.0.3 | Frontend bundler | Tauri default, fast HMR |
| Tailwind CSS | 4.2.2 | Utility-first styling | CSS-first config (@theme), shadcn/ui v4 built for it |
| @tailwindcss/vite | 4.2.2 | Tailwind Vite plugin | Required for Tailwind 4 in Vite |
| shadcn/ui | CLI v4 | Component library | Copies components into project, Tailwind 4 + React 19 native |
| Zustand | 5.0.12 | Client-side state | Minimal boilerplate, slices pattern for IDE state |
| TanStack Router | 1.168.7 | File-based routing | Type-safe, Vite plugin auto-generates route tree |
| @tanstack/router-plugin | 1.167.8 | Vite plugin for TanStack Router | Auto route tree generation |
| TanStack Query | 5.95.2 | Async state (Tauri command wrappers) | Caching, deduplication for invoke() calls |
| Drizzle ORM | 0.45.2 | Type-safe SQL (sqlite-proxy) | Proxy mode purpose-built for Tauri WebView |
| drizzle-kit | 0.31.10 | Migration generation | `drizzle-kit generate` for SQLite migrations |
| lucide-react | 1.7.0 | Icons | shadcn/ui standard icon library |
| geist | 1.7.0 | Geist Sans + Mono fonts | npm package bundles both font families |
| Biome | 2.4.9 | Linting + formatting | Single binary, 25x faster than ESLint+Prettier |
| Vitest | 4.1.2 | Unit + integration tests | Vite-native, Jest-compatible API |
| Turborepo | 2.8.x | Monorepo orchestration | Incremental builds, task graph, pnpm native |

### Rust Crates (Cargo.toml)

| Crate | Version | Purpose |
|-------|---------|---------|
| tauri | 2.10.3 | Core framework |
| tauri-plugin-sql | 2.3.2 | SQLite access via sqlx |
| tauri-plugin-log | 2.8.0 | Structured logging + log forwarding |
| serde | 1.0.228 | Serialization/deserialization |
| serde_json | 1.0.149 | JSON handling |
| tokio | 1.50.0 | Async runtime (required by Tauri) |
| tracing | 0.1.44 | Structured logging |
| tracing-subscriber | 0.3.23 | Log subscriber with formatting |
| uuid | 1.23.0 | UUID generation for project IDs |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| TanStack Router | React Router v7 | React Router lacks file-based routing with type-safe codegen; TanStack Router is the better fit for an IDE app with many routes |
| Drizzle sqlite-proxy | tauri-plugin-sql directly | Direct plugin use means raw SQL strings in frontend with no type safety; Drizzle adds zero runtime overhead via proxy |
| Zustand | Jotai | Jotai atomic model better for forms, not complex interconnected IDE state (panel layout, active project, IPC state) |
| Biome | ESLint + Prettier | 6 packages to configure vs 1 binary; monorepo speed advantage is material |

### Installation

```bash
# 1. Root monorepo (from empty directory)
pnpm init
pnpm add -Dw turbo typescript

# 2. Scaffold Tauri app in apps/desktop
pnpm create tauri-app apps/desktop --template react-ts --manager pnpm

# 3. Frontend deps (in apps/desktop)
cd apps/desktop
pnpm add react@^19.2 react-dom@^19.2
pnpm add @tauri-apps/api@^2.10 @tauri-apps/plugin-sql@^2.3 @tauri-apps/plugin-process@^2.3
pnpm add tailwindcss@^4.2 @tailwindcss/vite@^4.2
pnpm add zustand@^5.0
pnpm add @tanstack/react-router@^1.168 @tanstack/react-router-devtools@^1.166
pnpm add @tanstack/react-query@^5.95
pnpm add drizzle-orm@^0.45
pnpm add lucide-react@^1.7 geist@^1.7
pnpm add -D @tanstack/router-plugin@^1.167
pnpm add -D drizzle-kit@^0.31
pnpm add -D @biomejs/biome@^2.4
pnpm add -D vitest@^4.1 @testing-library/react
pnpm add -D vite@^8.0 @vitejs/plugin-react
pnpm add -D typescript@^6.0
```

```toml
# apps/desktop/src-tauri/Cargo.toml
[dependencies]
tauri = { version = "2.10", features = ["tray-icon"] }
tauri-plugin-sql = { version = "2.3", features = ["sqlite"] }
tauri-plugin-log = "2.8"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tokio = { version = "1", features = ["full"] }
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["json"] }
uuid = { version = "1", features = ["v4", "serde"] }

[build-dependencies]
tauri-build = { version = "2.10", features = [] }
```

## Architecture Patterns

### Recommended Project Structure

```
ultrabuilder/
  apps/
    desktop/
      src/
        components/
          layout/
            AppShell.tsx           # VS Code-style layout container
            ActivityBar.tsx        # Far-left icon bar
            Sidebar.tsx            # Collapsible sidebar
            EditorArea.tsx         # Main content area
            BottomPanel.tsx        # Log, IPC Debug, Problems tabs
            Titlebar.tsx           # Custom titlebar (drag region)
          projects/
            ProjectList.tsx        # List view
            ProjectCard.tsx        # Individual project card
            CreateProjectDialog.tsx
          debug/
            IpcDebugPanel.tsx      # Dev-only IPC event viewer
        hooks/
          use-tauri-command.ts     # TanStack Query + invoke wrapper
          use-tauri-channel.ts     # Channel listener hook
          use-tauri-event.ts       # Event listener hook
        lib/
          db.ts                    # Drizzle sqlite-proxy instance
          ipc.ts                   # Typed invoke wrappers
        routes/
          __root.tsx               # Root layout (AppShell)
          index.tsx                # Redirects to /projects
          projects/
            index.tsx              # Projects list
            $projectId.tsx         # Single project view
        stores/
          ui-store.ts             # Panel state, theme, sidebar open/closed
          projects-store.ts       # Active project context
          ipc-store.ts            # IPC debug event buffer (dev only)
        styles/
          globals.css             # Tailwind @import, @theme, Geist fonts
        App.tsx                   # Router provider + QueryClient provider
        main.tsx                  # Entry point
      src-tauri/
        src/
          main.rs                 # App setup, plugin registration, menu
          lib.rs                  # Tauri command handler registration
          state.rs                # AppState definition (Mutex-wrapped)
          commands/
            mod.rs
            projects.rs           # CRUD: create, list, update, delete
            sql.rs                # run_sql proxy command for Drizzle
          logging/
            mod.rs
            channel_layer.rs      # tracing Layer that sends to Channel
        migrations/               # SQL migration files (committed)
        tauri.conf.json
        Cargo.toml
        capabilities/
          default.json            # Permissions for plugins + window
      index.html
      vite.config.ts
      tsconfig.json
      package.json
  packages/
    shared/
      src/
        schema.ts                 # Drizzle table definitions
        types.ts                  # TypeScript types generated from schema
        index.ts                  # Re-exports
      package.json
      tsconfig.json
  turbo.json
  pnpm-workspace.yaml
  package.json                    # Root: packageManager, scripts
  biome.json
  .github/
    workflows/
      ci.yml                     # Cross-platform build + test
```

### Pattern 1: Drizzle SQLite Proxy for Tauri

**What:** Frontend computes SQL via Drizzle, sends to Rust backend for execution via tauri-plugin-sql.
**When to use:** All database operations in the Tauri WebView.

**TypeScript side (lib/db.ts):**
```typescript
// Source: https://dev.to/huakun/drizzle-sqlite-in-tauri-app-kif
import { drizzle } from "drizzle-orm/sqlite-proxy";
import { invoke } from "@tauri-apps/api/core";
import * as schema from "@ultrabuilder/shared/schema";

export const db = drizzle<typeof schema>(
  async (sql, params, method) => {
    const result = await invoke<any>("run_sql", {
      sql,
      params,
      method, // "all" | "get" | "values" | "run"
    });
    return { rows: result };
  },
  { schema }
);
```

**Rust side (commands/sql.rs):**
```rust
// Source: https://github.com/tdwesten/tauri-drizzle-sqlite-proxy-demo
use tauri::State;
use tauri_plugin_sql::{DbInstances, DbPool};
use serde_json::Value;

#[tauri::command]
pub async fn run_sql(
    db: State<'_, DbInstances>,
    sql: String,
    params: Vec<Value>,
    method: String,
) -> Result<Value, String> {
    let pool = db.0.read().await;
    let db_pool = pool.get("sqlite:ultrabuilder.db")
        .ok_or("Database not found")?;

    match db_pool {
        DbPool::Sqlite(pool) => {
            let mut query = sqlx::query(&sql);
            for param in &params {
                query = match param {
                    Value::String(s) => query.bind(s.as_str()),
                    Value::Number(n) => {
                        if let Some(i) = n.as_i64() { query.bind(i) }
                        else if let Some(f) = n.as_f64() { query.bind(f) }
                        else { query }
                    }
                    Value::Null => query.bind(Option::<String>::None),
                    Value::Bool(b) => query.bind(*b),
                    _ => query,
                };
            }
            // Execute based on method
            match method.as_str() {
                "all" | "get" | "values" => {
                    let rows = query.fetch_all(pool).await
                        .map_err(|e| e.to_string())?;
                    // Convert rows to JSON array
                    Ok(serde_json::to_value(rows).unwrap_or(Value::Array(vec![])))
                }
                "run" => {
                    query.execute(pool).await
                        .map_err(|e| e.to_string())?;
                    Ok(Value::Null)
                }
                _ => Err(format!("Unknown method: {}", method)),
            }
        }
        #[allow(unreachable_patterns)]
        _ => Err("Only SQLite is supported".into()),
    }
}
```

**Migration on startup (main.rs):**
```rust
// Run Drizzle-generated SQL migrations at app startup
use std::fs;

fn run_migrations(app: &tauri::App) -> Result<(), Box<dyn std::error::Error>> {
    let migrations_dir = app.path().resource_dir()?.join("migrations");
    // Read and execute .sql files in order
    // tauri-plugin-sql handles the connection
    Ok(())
}
```

**Drizzle schema (packages/shared/src/schema.ts):**
```typescript
import { sqliteTable, text, integer } from "drizzle-orm/sqlite-core";

export const projects = sqliteTable("projects", {
  id: text("id").primaryKey(), // UUID v4
  name: text("name").notNull(),
  description: text("description").notNull().default(""),
  githubRepo: text("github_repo"),
  status: text("status", { enum: ["planning", "building", "complete"] })
    .notNull()
    .default("planning"),
  createdAt: integer("created_at", { mode: "timestamp" })
    .notNull()
    .$defaultFn(() => new Date()),
  updatedAt: integer("updated_at", { mode: "timestamp" })
    .notNull()
    .$defaultFn(() => new Date()),
});

export type Project = typeof projects.$inferSelect;
export type NewProject = typeof projects.$inferInsert;
```

### Pattern 2: Tauri Command + Channel Pair

**What:** Command initiates an operation; Channel streams data back to frontend.
**When to use:** Log streaming (D-10), any long-running operation.

**Rust side:**
```rust
// Source: https://docs.rs/tauri/latest/tauri/ipc/struct.Channel.html
use tauri::ipc::Channel;
use serde::Serialize;

#[derive(Clone, Serialize)]
#[serde(rename_all = "camelCase", tag = "event", content = "data")]
pub enum LogEvent {
    #[serde(rename_all = "camelCase")]
    LogMessage {
        level: String,
        target: String,
        message: String,
        timestamp: i64,
    },
}

#[tauri::command]
pub async fn start_log_stream(on_event: Channel<LogEvent>) -> Result<(), String> {
    // Send log events through the channel
    on_event.send(LogEvent::LogMessage {
        level: "info".into(),
        target: "app::startup".into(),
        message: "Application started".into(),
        timestamp: chrono::Utc::now().timestamp_millis(),
    }).map_err(|e| e.to_string())?;

    Ok(())
}
```

**TypeScript side:**
```typescript
import { invoke, Channel } from "@tauri-apps/api/core";

interface LogEvent {
  event: "logMessage";
  data: {
    level: string;
    target: string;
    message: string;
    timestamp: number;
  };
}

export async function startLogStream(
  onEvent: (event: LogEvent) => void
): Promise<void> {
  const channel = new Channel<LogEvent>();
  channel.onmessage = onEvent;
  await invoke("start_log_stream", { onEvent: channel });
}
```

### Pattern 3: Tauri Events (Lifecycle)

**What:** Fire-and-forget broadcasts for app lifecycle state changes.
**When to use:** D-11 lifecycle events (app:ready, window:focus, window:blur).

**Rust side (emit):**
```rust
use tauri::Emitter;

// In app setup:
app.emit("app:ready", ()).unwrap();

// In window event handler:
app.listen("tauri://focus", |_event| {
    app.emit("window:focus", ()).unwrap();
});
app.listen("tauri://blur", |_event| {
    app.emit("window:blur", ()).unwrap();
});
```

**TypeScript side (listen):**
```typescript
import { listen } from "@tauri-apps/api/event";

// In a React hook or effect:
const unlisten = await listen("app:ready", () => {
  useUiStore.getState().setAppReady(true);
});

const unlistenFocus = await listen("window:focus", () => {
  useUiStore.getState().setWindowFocused(true);
});
```

### Pattern 4: Custom Titlebar with Drag Region

**What:** Hide native decorations, render custom titlebar in HTML/CSS.
**When to use:** D-13 custom titlebar requirement.

**tauri.conf.json:**
```json
{
  "app": {
    "windows": [
      {
        "title": "Ultrabuilder",
        "width": 1280,
        "height": 800,
        "minWidth": 900,
        "minHeight": 600,
        "decorations": false,
        "transparent": false
      }
    ]
  }
}
```

**Titlebar.tsx:**
```typescript
export function Titlebar() {
  return (
    <div
      data-tauri-drag-region
      className="h-9 flex items-center justify-between bg-background border-b select-none"
    >
      {/* macOS traffic light spacer (left) */}
      <div className="w-[70px] macos-only" />

      <span data-tauri-drag-region className="text-xs font-medium text-muted-foreground">
        Ultrabuilder
      </span>

      {/* Windows/Linux window controls (right) */}
      <div className="flex items-center windows-linux-only">
        <WindowMinimizeButton />
        <WindowMaximizeButton />
        <WindowCloseButton />
      </div>
    </div>
  );
}
```

**Capabilities (capabilities/default.json):**
```json
{
  "identifier": "default",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "core:window:default",
    "core:window:allow-start-dragging",
    "core:window:allow-minimize",
    "core:window:allow-maximize",
    "core:window:allow-unmaximize",
    "core:window:allow-close",
    "core:window:allow-set-title",
    "sql:default",
    "sql:allow-execute",
    "sql:allow-select",
    "process:default",
    "log:default"
  ]
}
```

### Pattern 5: Tauri Menu API (Rust)

**What:** Native application menu bar built in Rust.
**When to use:** D-14 full IDE menu (File, Edit, View, Window, Help).

```rust
// Source: https://v2.tauri.app/learn/window-menu/
use tauri::menu::{Menu, Submenu, MenuItem, PredefinedMenuItem};

fn create_menu(app: &tauri::App) -> Result<Menu<tauri::Wry>, tauri::Error> {
    let menu = Menu::with_items(app, &[
        &Submenu::with_items(app, "File", true, &[
            &MenuItem::with_id(app, "new-project", "New Project", true, Some("CmdOrCtrl+N"))?,
            &MenuItem::with_id(app, "open-project", "Open Project", true, Some("CmdOrCtrl+O"))?,
            &PredefinedMenuItem::separator(app)?,
            &PredefinedMenuItem::quit(app, Some("Quit"))?,
        ])?,
        &Submenu::with_items(app, "Edit", true, &[
            &PredefinedMenuItem::undo(app, Some("Undo"))?,
            &PredefinedMenuItem::redo(app, Some("Redo"))?,
            &PredefinedMenuItem::separator(app)?,
            &PredefinedMenuItem::cut(app, Some("Cut"))?,
            &PredefinedMenuItem::copy(app, Some("Copy"))?,
            &PredefinedMenuItem::paste(app, Some("Paste"))?,
            &PredefinedMenuItem::select_all(app, Some("Select All"))?,
        ])?,
        &Submenu::with_items(app, "View", true, &[
            &MenuItem::with_id(app, "toggle-sidebar", "Toggle Sidebar", true, Some("CmdOrCtrl+B"))?,
            &MenuItem::with_id(app, "toggle-bottom-panel", "Toggle Bottom Panel", true, Some("CmdOrCtrl+J"))?,
            &PredefinedMenuItem::separator(app)?,
            &MenuItem::with_id(app, "zoom-in", "Zoom In", true, Some("CmdOrCtrl+Plus"))?,
            &MenuItem::with_id(app, "zoom-out", "Zoom Out", true, Some("CmdOrCtrl+Minus"))?,
            &MenuItem::with_id(app, "zoom-reset", "Reset Zoom", true, Some("CmdOrCtrl+0"))?,
        ])?,
        &Submenu::with_items(app, "Window", true, &[
            &PredefinedMenuItem::minimize(app, Some("Minimize"))?,
            &PredefinedMenuItem::maximize(app, Some("Zoom"))?,
            &PredefinedMenuItem::close_window(app, Some("Close"))?,
        ])?,
        &Submenu::with_items(app, "Help", true, &[
            &MenuItem::with_id(app, "about", "About Ultrabuilder", true, None::<&str>)?,
        ])?,
    ])?;

    Ok(menu)
}
```

**macOS note:** The first submenu is always placed under the application menu by default. Add an app-name submenu as the first entry on macOS, or accept that "File" becomes the about menu.

### Pattern 6: TanStack Router File-Based Setup

**What:** File-based routing with Vite plugin for auto route tree generation.
**When to use:** D-04 TanStack Router requirement.

**vite.config.ts:**
```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";
import { TanStackRouterVite } from "@tanstack/router-plugin/vite";
import path from "path";

export default defineConfig({
  plugins: [
    TanStackRouterVite({ target: "react", autoCodeSplitting: true }),
    react(),
    tailwindcss(),
  ],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  // Tauri expects a fixed port for dev
  server: {
    port: 1420,
    strictPort: true,
  },
  // Tauri expects process.env variables
  envPrefix: ["VITE_", "TAURI_"],
});
```

### Pattern 7: turbo.json Task Graph

**What:** Turborepo task configuration for the monorepo.

```json
{
  "$schema": "https://turborepo.dev/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "src-tauri/target/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["^build"],
      "cache": false
    },
    "lint": {},
    "check": {
      "dependsOn": ["^build"]
    },
    "tauri:dev": {
      "cache": false,
      "persistent": true,
      "dependsOn": ["^build"]
    },
    "tauri:build": {
      "dependsOn": ["^build"],
      "outputs": ["src-tauri/target/release/bundle/**"]
    }
  }
}
```

**pnpm-workspace.yaml:**
```yaml
packages:
  - "apps/*"
  - "packages/*"
```

**Root package.json:**
```json
{
  "name": "ultrabuilder",
  "private": true,
  "packageManager": "pnpm@9.15.0",
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "check": "turbo run check"
  },
  "devDependencies": {
    "turbo": "^2.8",
    "typescript": "^6.0"
  }
}
```

### Pattern 8: Zustand Slice Pattern

**What:** One Zustand store per domain with typed selectors.

```typescript
// stores/ui-store.ts
import { create } from "zustand";

interface UiState {
  sidebarOpen: boolean;
  bottomPanelOpen: boolean;
  bottomPanelTab: "log" | "ipc-debug" | "problems";
  theme: "light" | "dark" | "system";
  appReady: boolean;
  windowFocused: boolean;

  toggleSidebar: () => void;
  toggleBottomPanel: () => void;
  setBottomPanelTab: (tab: UiState["bottomPanelTab"]) => void;
  setAppReady: (ready: boolean) => void;
  setWindowFocused: (focused: boolean) => void;
}

export const useUiStore = create<UiState>((set) => ({
  sidebarOpen: true,
  bottomPanelOpen: true,
  bottomPanelTab: "log",
  theme: "system",
  appReady: false,
  windowFocused: true,

  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
  toggleBottomPanel: () => set((s) => ({ bottomPanelOpen: !s.bottomPanelOpen })),
  setBottomPanelTab: (tab) => set({ bottomPanelTab: tab }),
  setAppReady: (ready) => set({ appReady: ready }),
  setWindowFocused: (focused) => set({ windowFocused: focused }),
}));
```

### Anti-Patterns to Avoid

- **Direct raw SQL in frontend:** Use Drizzle ORM exclusively. Never call tauri-plugin-sql directly from React components.
- **Events for streaming:** Use Channels for log streaming (D-10). Events are for fire-and-forget lifecycle notifications only.
- **Monolithic lib.rs:** Split Rust code into modules from day one (commands/, logging/). Keep files under 500 lines per CLAUDE.md.
- **Manual route definitions:** Use TanStack Router file-based routing. Never hand-write route trees.
- **CSS-in-JS for theming:** Use Tailwind 4 @theme directive + CSS custom properties. shadcn/ui handles dark mode via class toggle.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| SQLite access from WebView | Custom fetch-based SQL proxy | Drizzle sqlite-proxy + tauri-plugin-sql | Battle-tested pattern, type-safe, handles edge cases (NULL binding, blob types) |
| Application menu | HTML/CSS fake menu | Tauri Menu API (Rust) | Native menu respects OS conventions, keyboard shortcuts work automatically |
| Window drag region | Custom mouse event handlers | `data-tauri-drag-region` attribute | Tauri handles platform-specific drag behavior internally |
| Dark/light theme | Custom theme context + localStorage | `prefers-color-scheme` media query + shadcn `dark` class | OS-native preference detection, zero JS overhead |
| Route code splitting | Manual React.lazy() | TanStack Router `autoCodeSplitting` | Automatic per-route splitting with type safety |
| UUID generation | JS uuid library in frontend | Rust `uuid` crate in commands | Generate IDs on the Rust side where the DB lives; no round-trip for ID creation |
| Cross-platform CI | Custom shell scripts per OS | tauri-apps/tauri-action | Handles platform deps, Rust toolchain, signing, artifact paths |

## Common Pitfalls

### Pitfall 1: WebView Divergence Across Platforms
**What goes wrong:** CSS/JS features work on macOS (WebKit) but break on Linux (WebKit GTK) or behave differently on Windows (WebView2/Chromium).
**Why it happens:** Tauri uses OS-native WebViews with different engine versions and feature support.
**How to avoid:** Set up cross-platform CI from day one (FNDN-04). Test the custom titlebar, drag region, and theme toggle on all three platforms in CI. Stick to ES2021 baseline.
**Warning signs:** Layout shifts on Windows vs macOS, drag region not working on Linux, theme toggle not respecting OS preference on one platform.

### Pitfall 2: Drizzle Migration Timing
**What goes wrong:** Migrations run before tauri-plugin-sql has initialized the database connection, causing "database not found" errors.
**Why it happens:** Tauri plugin initialization is async and happens during `app.setup()`. If migration code runs too early, the DB pool is empty.
**How to avoid:** Run migrations inside the `setup` callback AFTER plugin registration. Use `app.state::<DbInstances>()` to verify the connection exists before migrating.
**Warning signs:** Intermittent startup crashes, "no such table" errors on first launch.

### Pitfall 3: macOS Custom Titlebar + Traffic Lights
**What goes wrong:** When `decorations: false` on macOS, the native traffic light buttons (close/minimize/maximize) disappear entirely. Users expect them.
**Why it happens:** `decorations: false` removes ALL native window chrome, including traffic lights.
**How to avoid:** On macOS, consider `titleBarStyle: "overlay"` or `hiddenTitle: true` instead of `decorations: false` to keep traffic lights. Alternatively, implement custom window control buttons for all platforms (VS Code approach). Test on macOS specifically.
**Warning signs:** macOS users cannot close/minimize/maximize via expected button positions.

### Pitfall 4: TanStack Router + Tauri Deep Links
**What goes wrong:** The router expects browser-like URL handling, but Tauri serves from a custom protocol (`tauri://localhost`). Hash-based routing or custom protocol handling may be needed.
**How to avoid:** Use TanStack Router in hash mode or configure Vite's base URL for Tauri. Test that route navigation works after app restart (no blank screen).
**Warning signs:** Blank screen after navigating to a route and restarting the app. 404 errors in the WebView console.

### Pitfall 5: pnpm Workspace Hoisting + Tauri
**What goes wrong:** Tauri's frontend build expects dependencies in `node_modules` at the app level, but pnpm hoists differently than npm.
**How to avoid:** Ensure `.npmrc` has `shamefully-hoist=true` or configure `public-hoist-pattern` for Tauri-specific packages. Verify `pnpm turbo build` works from a clean install.
**Warning signs:** "Module not found" errors during `tauri build` that don't appear during `vite dev`.

### Pitfall 6: Channel Cleanup on Component Unmount
**What goes wrong:** Channel listeners are not cleaned up when React components unmount, causing memory leaks and state updates on unmounted components.
**Why it happens:** Tauri Channels don't have a built-in `unlisten` like Events. The Channel object must be dropped.
**How to avoid:** Wrap Channel creation in a custom hook with useEffect cleanup. Set a flag to ignore messages after unmount. Use AbortController pattern if available.
**Warning signs:** Console warnings about state updates on unmounted components, memory growing over time.

## Code Examples

### Geist Font Setup (globals.css)

```css
/* Source: geist npm package */
@import "tailwindcss";
@import "geist/font/sans.css";
@import "geist/font/mono.css";

@theme {
  --font-sans: "Geist Sans", ui-sans-serif, system-ui, sans-serif;
  --font-mono: "Geist Mono", ui-monospace, monospace;
}
```

### TanStack Query + Tauri Command Hook

```typescript
// hooks/use-tauri-command.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { invoke } from "@tauri-apps/api/core";
import type { Project, NewProject } from "@ultrabuilder/shared";

export function useProjects() {
  return useQuery({
    queryKey: ["projects"],
    queryFn: () => invoke<Project[]>("list_projects"),
  });
}

export function useCreateProject() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (project: NewProject) =>
      invoke<Project>("create_project", { project }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["projects"] });
    },
  });
}
```

### Rust Command Handlers (commands/projects.rs)

```rust
use crate::state::AppState;
use serde::{Deserialize, Serialize};
use tauri::State;
use tokio::sync::Mutex;
use uuid::Uuid;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Project {
    pub id: String,
    pub name: String,
    pub description: String,
    pub github_repo: Option<String>,
    pub status: String,
    pub created_at: i64,
    pub updated_at: i64,
}

#[derive(Debug, Deserialize)]
pub struct NewProject {
    pub name: String,
    pub description: Option<String>,
}

#[tauri::command]
pub async fn create_project(project: NewProject) -> Result<Project, String> {
    let now = chrono::Utc::now().timestamp();
    let p = Project {
        id: Uuid::new_v4().to_string(),
        name: project.name,
        description: project.description.unwrap_or_default(),
        github_repo: None,
        status: "planning".into(),
        created_at: now,
        updated_at: now,
    };
    // DB insert via run_sql (called from frontend Drizzle, not here)
    // This command pattern is for direct Rust-side CRUD
    Ok(p)
}

#[tauri::command]
pub async fn list_projects() -> Result<Vec<Project>, String> {
    // Query via tauri-plugin-sql
    Ok(vec![])
}
```

### GitHub Actions CI (`.github/workflows/ci.yml`)

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        include:
          - platform: macos-latest
            rust-target: aarch64-apple-darwin
          - platform: macos-latest
            rust-target: x86_64-apple-darwin
          - platform: ubuntu-22.04
            rust-target: x86_64-unknown-linux-gnu
          - platform: windows-latest
            rust-target: x86_64-pc-windows-msvc
    runs-on: ${{ matrix.platform }}

    steps:
      - uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: pnpm

      - name: Install Rust stable
        uses: dtolnay/rust-toolchain@stable
        with:
          targets: ${{ matrix.rust-target }}

      - name: Rust cache
        uses: swatinem/rust-cache@v2
        with:
          workspaces: apps/desktop/src-tauri -> target

      - name: Install Linux dependencies
        if: matrix.platform == 'ubuntu-22.04'
        run: |
          sudo apt-get update
          sudo apt-get install -y \
            libwebkit2gtk-4.1-dev \
            libappindicator3-dev \
            librsvg2-dev \
            patchelf

      - name: Install frontend dependencies
        run: pnpm install --frozen-lockfile

      - name: Lint
        run: pnpm turbo lint

      - name: Test (frontend)
        run: pnpm turbo test

      - name: Build packages
        run: pnpm turbo build --filter=@ultrabuilder/shared

      - name: Build Tauri app
        uses: tauri-apps/tauri-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          projectPath: apps/desktop
          args: --target ${{ matrix.rust-target }}
          tauriScript: pnpm tauri
```

### IPC Debug Panel (Dev Only)

```typescript
// components/debug/IpcDebugPanel.tsx
import { useIpcStore } from "@/stores/ipc-store";

interface IpcEvent {
  id: number;
  timestamp: number;
  type: "command" | "channel" | "event";
  direction: "invoke" | "response" | "emit" | "listen";
  name: string;
  payload: unknown;
}

export function IpcDebugPanel() {
  if (import.meta.env.PROD) return null;

  const events = useIpcStore((s) => s.events);

  return (
    <div className="h-full overflow-auto font-mono text-xs">
      <table className="w-full">
        <thead className="sticky top-0 bg-background">
          <tr className="text-left text-muted-foreground">
            <th className="px-2 py-1 w-24">Time</th>
            <th className="px-2 py-1 w-16">Type</th>
            <th className="px-2 py-1 w-12">Dir</th>
            <th className="px-2 py-1 w-40">Name</th>
            <th className="px-2 py-1">Payload</th>
          </tr>
        </thead>
        <tbody>
          {events.map((event) => (
            <tr key={event.id} className="border-t border-border hover:bg-muted/50">
              <td className="px-2 py-0.5">{formatTimestamp(event.timestamp)}</td>
              <td className="px-2 py-0.5">{event.type}</td>
              <td className="px-2 py-0.5">{event.direction === "invoke" ? "->" : "<-"}</td>
              <td className="px-2 py-0.5">{event.name}</td>
              <td className="px-2 py-0.5 truncate max-w-[300px]">
                {JSON.stringify(event.payload)}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Tailwind v3 config (tailwind.config.js) | Tailwind v4 CSS-first (@theme directive, @import "tailwindcss") | Jan 2025 | No JS config file needed, use @tailwindcss/vite plugin |
| shadcn/ui v3 (Radix primitives) | shadcn/ui v4 (Base UI primitives, RTL, app blocks) | March 2026 | CLI v4 generates Tailwind 4 compatible components |
| Tauri v1 events for everything | Tauri v2 Commands + Channels + Events (three-tier) | Oct 2024 | Channels are purpose-built for streaming, Events for lifecycle only |
| drizzle-kit push | drizzle-kit generate + migrate | 2024-2025 | Generate SQL migration files, commit to repo, run on startup |
| TanStack Router code-based | TanStack Router file-based (Vite plugin) | 2024 | Auto route tree generation, type-safe params |

**Deprecated/outdated:**
- `tauri::api::event` (v1) replaced by `tauri::Emitter` trait (v2)
- Tailwind v3 `tailwind.config.js` replaced by CSS-first `@theme` in v4
- shadcn/ui `components.json` format changed in v4 CLI

## Open Questions

1. **Drizzle migration execution on Rust side**
   - What we know: Drizzle generates `.sql` migration files. tauri-plugin-sql can execute raw SQL.
   - What's unclear: The exact mechanism to read migration files from the app bundle and execute them in order on first launch. Need to handle migration tracking (which migrations have run).
   - Recommendation: Implement a simple `_migrations` table that tracks applied migrations by filename. On startup, compare migration files in bundle vs applied migrations, run any unapplied ones in order.

2. **macOS traffic lights with decorations: false**
   - What we know: `decorations: false` removes traffic lights entirely. Some Tauri apps use `titleBarStyle: "overlay"` to keep them.
   - What's unclear: Whether `titleBarStyle` is available in Tauri 2.10 config or only via Rust API.
   - Recommendation: Investigate at implementation time. Fallback is custom window control buttons on all platforms.

3. **Tauri Channel lifecycle management**
   - What we know: Channels are created in JS and passed to Rust commands. They stream data back.
   - What's unclear: How to properly "close" a Channel from the Rust side when the operation completes, and how React cleanup interacts with this.
   - Recommendation: Use a terminal event pattern (send a "done" event through the Channel) and handle cleanup in useEffect return.

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | Vitest 4.1.2 |
| Config file | `apps/desktop/vitest.config.ts` (Wave 0) |
| Quick run command | `pnpm turbo test --filter=@ultrabuilder/desktop` |
| Full suite command | `pnpm turbo test` |

### Phase Requirements to Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| FNDN-01 | Tauri app launches with React UI on 3 platforms | CI matrix build | `tauri-apps/tauri-action` in GitHub Actions (build succeeds = pass) | Wave 0 |
| FNDN-02 | Monorepo builds all packages with single command | integration | `pnpm turbo build` (exit code 0 = pass) | Wave 0 |
| FNDN-03 | SQLite persists project data across restarts | unit | `vitest run --filter=db` -- test create/read/update/delete via Drizzle proxy | Wave 0 |
| FNDN-04 | CI pipeline runs on all 3 OS targets | CI workflow | `.github/workflows/ci.yml` matrix completes green | Wave 0 |
| FNDN-05a | Commands: project CRUD works | unit | `vitest run --filter=commands` -- test invoke wrappers return correct data | Wave 0 |
| FNDN-05b | Channels: log streaming delivers events | integration | `vitest run --filter=channel` -- test Channel receives events in order | Wave 0 |
| FNDN-05c | Events: lifecycle events fire correctly | integration | `vitest run --filter=events` -- test event listeners receive payloads | Wave 0 |

### Sampling Rate
- **Per task commit:** `pnpm turbo test --filter=@ultrabuilder/desktop`
- **Per wave merge:** `pnpm turbo test && pnpm turbo lint && pnpm turbo build`
- **Phase gate:** Full suite green + CI matrix green on all 3 platforms before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] `apps/desktop/vitest.config.ts` -- Vitest configuration for the desktop app
- [ ] `apps/desktop/src/__tests__/db.test.ts` -- Drizzle proxy CRUD tests (mocked invoke)
- [ ] `apps/desktop/src/__tests__/stores.test.ts` -- Zustand store unit tests
- [ ] `packages/shared/src/__tests__/schema.test.ts` -- Schema type validation
- [ ] `.github/workflows/ci.yml` -- Cross-platform CI workflow
- [ ] `apps/desktop/src/test-utils.ts` -- Mock invoke/Channel/listen helpers for Vitest

**Note on Tauri IPC testing:** Vitest runs in Node.js, not in the Tauri WebView. Tauri commands (`invoke`, `Channel`, `listen`) must be mocked for unit tests. Integration tests that exercise real IPC require `tauri-driver` or Playwright with a running Tauri app -- these are CI-only tests, not quick-run tests.

## Sources

### Primary (HIGH confidence)
- [Tauri 2.0 official docs](https://v2.tauri.app/) -- IPC concepts, calling Rust, window customization, menu API, configuration, GitHub Actions pipeline
- [Tauri 2.0 IPC Channel API](https://docs.rs/tauri/latest/tauri/ipc/struct.Channel.html) -- Rust Channel struct documentation
- [Tauri Window Customization](https://v2.tauri.app/learn/window-customization/) -- decorations: false, drag region, platform differences
- [Tauri Menu API](https://v2.tauri.app/learn/window-menu/) -- Rust menu builder, PredefinedMenuItem, macOS quirks
- [TanStack Router Vite installation](https://tanstack.com/router/latest/docs/framework/react/quick-start) -- File-based routing setup
- [Drizzle + SQLite in Tauri](https://dev.to/huakun/drizzle-sqlite-in-tauri-app-kif) -- Proxy pattern walkthrough
- [tauri-drizzle-sqlite-proxy-demo](https://github.com/tdwesten/tauri-drizzle-sqlite-proxy-demo) -- Working reference implementation
- [Tauri GitHub Actions pipeline](https://v2.tauri.app/distribute/pipelines/github/) -- Official CI/CD workflow
- [tauri-apps/tauri-action](https://github.com/tauri-apps/tauri-action) -- Cross-platform build action
- [shadcn/ui Vite installation](https://ui.shadcn.com/docs/installation/vite) -- Tailwind 4 + Vite setup
- [Turborepo configuration](https://turborepo.dev/docs/reference/configuration) -- turbo.json task graph
- npm registry -- all package versions verified 2026-03-28
- crates.io -- all Rust crate versions verified 2026-03-28

### Secondary (MEDIUM confidence)
- [Drizzle SQLite Migrations in Tauri 2.0](https://keypears.com/blog/2025-10-04-drizzle-sqlite-tauri) -- Migration strategy
- [Tauri custom menu items blog](https://ratulmaharaj.com/posts/tauri-custom-menu/) -- Practical menu examples
- [Tauri v2 GitHub Actions release guide](https://dev.to/tomtomdu73/ship-your-tauri-v2-app-like-a-pro-github-actions-and-release-automation-part-22-2ef7) -- CI/CD patterns

### Tertiary (LOW confidence)
- macOS `titleBarStyle` availability in Tauri 2.10 config -- needs implementation-time verification

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- all versions verified against npm/crates.io registries on 2026-03-28
- Architecture: HIGH -- patterns sourced from official Tauri docs, working reference implementations, and prior project research
- Pitfalls: HIGH -- sourced from Tauri GitHub issues, community discussions, and prior PITFALLS.md research
- Validation: MEDIUM -- Tauri IPC mocking strategy needs validation at implementation time

**Research date:** 2026-03-28
**Valid until:** 2026-04-28 (stable ecosystem, 30-day window)
