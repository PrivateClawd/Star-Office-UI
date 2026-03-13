# Star Desktop Pet (Electron Shell)

This directory is the Electron version of the desktop shell, running in parallel with the existing Tauri version for a gradual migration path.

## Features

- Reuses the existing frontend: `http://127.0.0.1:19000/?desktop=1`
- Reuses the mini page: `desktop-pet/src/minimized.html`
- Auto-starts the Python backend on launch (if not already running)
- Main window / mini window toggle
- System tray (menu bar) persistent menu
- Injects a `window.__TAURI__` compatibility layer via preload to minimize changes to existing frontend logic

## Getting Started

```bash
cd electron-shell
npm install
npm run dev
```

## Optional environment variables

- `STAR_PROJECT_ROOT`: Project root directory (auto-detected by default)
- `STAR_BACKEND_PYTHON`: Backend Python executable path
- `STAR_BACKEND_HOST`: Backend host (default `127.0.0.1`)
- `STAR_BACKEND_PORT`: Backend port (default `19000`)

## Notes

- This is currently a "runnable migration skeleton" — the goal is to replace the desktop container layer first.
- The existing Tauri directory is unaffected; you can roll back or compare side by side at any time.
