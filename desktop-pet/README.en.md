# Star Office Tauri Desktop Shell

This directory packages `Star-Office-UI` as a desktop application (transparent window) that automatically starts the backend process on launch.

## Development

First, set up the Python environment in the repo root:

```bash
cd Star-Office-UI
uv venv .venv
uv pip install -r backend/requirements.txt --python .venv/bin/python
```

Then start Tauri:

```bash
cd desktop-pet
npm install
npm run dev
```

## Auto-start backend logic

- Preferred: `../.venv/bin/python backend/app.py`
- Fallback: `python3 backend/app.py`
- Last resort: `python backend/app.py`

The window opens by default at:

- `http://127.0.0.1:19000/?desktop=1`

## Optional environment variables

- `STAR_PROJECT_ROOT`: Project root directory (auto-detected by default)
- `STAR_BACKEND_PYTHON`: Custom Python executable path
- `STAR_BACKEND_URL`: Custom URL for the desktop window
