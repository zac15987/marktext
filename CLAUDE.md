# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MarkText is an open-source Electron-based markdown editor. It uses Vue.js 2 for the renderer UI and a custom markdown editing library called **Muya** (built on Snabbdom virtual DOM). The app targets Linux, macOS, and Windows.

## Common Commands

```bash
yarn run dev              # Run in development mode with hot reload
yarn run lint             # Lint JS/Vue files
yarn run lint:fix         # Auto-fix lint issues
yarn run unit             # Run unit tests (Karma + Mocha + Chai)
yarn run e2e              # Run E2E tests (Playwright) — requires `yarn run pack` first
yarn run test             # Run unit + E2E tests
yarn run test:specs       # Run CommonMark/GFM spec compliance tests
yarn run pack:main        # Build main process only
yarn run pack:renderer    # Build renderer process only
yarn run build:bin        # Build binary without installer
yarn run build            # Build full distribution packages
yarn run rebuild          # Rebuild native modules (electron-rebuild)
```

## Architecture

### Electron Two-Process Model

- **Main process** (`src/main/`): Node.js — manages windows, file I/O, menus, IPC handlers, settings persistence
- **Renderer process** (`src/renderer/`): Chromium + Vue.js — UI, editor state, user interaction
- **Shared code** (`src/common/`): Constants, filesystem utilities, keybinding helpers used by both processes

### Main Process (`src/main/`)

Entry point: `src/main/index.js`. Key classes:
- **App** (`app/index.js`) — application controller, lifecycle, IPC listeners
- **Accessor** (`app/accessor.js`) — dependency injection container for app modules
- **WindowManager** (`app/windowManager.js`) — manages EditorWindow and SettingWindow instances
- **DataCenter** (`dataCenter/`) — persistent data (settings, recent files) via electron-store

Subfolders: `cli/`, `commands/`, `contextMenu/`, `filesystem/`, `keyboard/`, `menu/`, `preferences/`, `spellchecker/`, `windows/`

### Renderer Process (`src/renderer/`)

Entry point: `src/renderer/main.js`. Bootstrap in `bootstrap.js` parses URL params and initializes globals.

- **Vuex store** (`store/`): `editor.js`, `layout.js`, `preferences.js`, `project.js`, `commandCenter.js`, `autoUpdates.js`, `notification.js`
- **Vue components** (`components/`): `editorWithTabs/`, `sideBar/`, `titleBar/`, `search/`
- **Services** (`services/`): business logic for file operations, export, search
- **Pages** (`pages/`): root Vue pages — `app.vue` (editor), `preference.vue` (settings)
- Uses Element UI for UI widgets and Vue Router for page routing

### Muya Editor Library (`src/muya/`)

Custom WYSIWYG/source markdown editor using Snabbdom virtual DOM. Has its own `package.json` (v0.1.2) and can be built independently with `yarn run build:muya`. Supports KaTeX, Mermaid, Vega, PrismJS syntax highlighting, and FlowChart.js.

### Build System

Webpack 5 with separate configs in `.electron-vue/`:
- `webpack.main.config.js` — main process bundle → `dist/electron/main.js`
- `webpack.renderer.config.js` — renderer bundle → `dist/electron/renderer.js`
- `dev-runner.js` — development server with hot reload
- `build.js` — production build script

Packaging via electron-builder (`electron-builder.yml`).

## Import Path Aliases

Defined in `.eslintrc.js` and `jsconfig.json` files:
- `@/` → `src/renderer/`
- `common/` → `src/common/`
- `muya/` → `src/muya/`

## Code Style

- ESLint Standard + Recommended, with Vue and HTML plugins
- **2-space indentation, no semicolons**
- ES6+ with `const`/`let` (no `var`)
- JSDoc for function documentation
- IPC channel names: `mt::kebab-case`
- Vuex mutations/actions: `SCREAMING_SNAKE_CASE`
- Vue components: PascalCase imports, kebab-case in templates

## Prerequisites

- Node.js `>=16` and `<17`, Yarn
- Python `>=3.6` (for node-gyp native module compilation)
- C++ compiler (Visual Studio 2019 on Windows)
- On Linux: `libx11-dev libxkbfile-dev libsecret-1-dev libfontconfig-dev`

## PR Conventions

- Target the `develop` branch
- Reference related issue numbers
- PR title for bug fixes: `fix: #<issue> <short message>`
- All CI checks and `yarn run lint` must pass
