# AGENTS.md

## Repo Purpose
This project builds a **single-page web app** that runs a **Python backend inside the user's browser (via Pyodide)** and exposes a small Django-based API layer that the frontend (Leaflet) calls when the user pans/zooms the map.  

The first dataset is **GeoTessera** (registry + .npy tiles). The architecture must remain flexible so we can later swap the data source to **AWS S3** or similar without changing the UI.

## Source of Truth
- `SPEC.md` is the authoritative spec. All deliverables and acceptance criteria live there.
- If something is ambiguous, follow `SPEC.md` first.

## Key Files / Structure
- `index.html` — main entry point, loads Pyodide and the UI.
- `app_py/backend.py` — Django setup + API routes + request bridge.
- `app_py/state.py` — AppState, registry cache, tile cache.
- `app_py/tiles.py` — bbox → tiles logic, tile → preview logic.
- `README.md` — how to run locally.

## Architecture Rules
- Use **Pyodide (pyodide.js)**, not PyScript.
- Backend runs fully in-browser.
- Frontend talks to backend through a JS → Pyodide bridge (e.g., `handle_request(path)`).
- Keep the **worker proxy fetch pattern** for remote data (CORS-safe).
- Keep GeoTessera registry loading + tile preview pipeline intact.
- Do **not** reinitialize Leaflet without removing the old map instance.

## What NOT to Break
- GeoTessera registry loading flow
- Worker proxy fetch flow
- Leaflet overlay rendering
- No “Map already initialized” errors

## Manual Test Procedure
1. Run a local server:
   ```bash
   python -m http.server
