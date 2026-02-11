
---

# 📄 `SPEC.md`

```markdown
# SPEC.md — Pyodide + Django + Leaflet Streaming Tile Explorer

## Goal
Build a **single-page web app** that runs a **Python backend inside the user's browser (Pyodide)** and uses it to serve/interpret API-like requests from the UI when the user pans/zooms a Leaflet map.

The app should:
- Stream tile/data from a remote source (start with GeoTessera registry + .npy tiles).
- Be structured so swapping to **AWS S3-hosted datasets** later is straightforward.
- Reuse the existing GeoTessera explorer/visualizer concepts: registry loading, proxy fetch, tile list, overlay visualization.

---

## Starting Point (Must Reuse Conceptually)
- GeoTessera Visualizer base:
  - Registry parquet download + caching
  - Proxy fetch pattern (worker proxy)
  - Tile selection list, year filter, overlay visualization
  - Leaflet overlay and footprint highlighting
- Existing proof-of-concept:
  - Django running in Pyodide
  - JS → Python bridge via something like `handle_request(path)`

---

## Deliverables

### 1. Project Structure
A working local web project with:
- `index.html` (Leaflet UI + Pyodide bootstrap)
- `app_py/backend.py` (Django setup + routes)
- `app_py/state.py` (AppState + caches)
- `app_py/tiles.py` (bbox → tiles, tile → preview)
- `README.md` (how to run locally, e.g. `python -m http.server`)

---

### 2. In-Browser Django Backend (Pyodide)

#### Routes
- `GET /api/health`
  - Returns simple JSON `{ "status": "ok" }`

- `GET /api/tiles?bbox=<west,south,east,north>&zoom=<z>&year=<year|all>&limit=<n>`
  - Returns JSON list of candidate tiles in/near the bbox.
  - Example response:
    ```json
    {
      "tiles": [
        {
          "tile_id": "grid_-118.25_34.05",
          "lat": 34.05,
          "lon": -118.25,
          "year": 2020,
          "bounds": [[34.0, -118.3], [34.1, -118.2]]
        }
      ]
    }
    ```

- `GET /api/tile/<tile_id>/preview`
  - Downloads the tile `.npy`
  - Converts to PNG data URI
  - Returns:
    ```json
    {
      "tile_id": "...",
      "bounds": [[...],[...]],
      "data_uri": "data:image/png;base64,...",
      "source_url": "https://..."
    }
    ```

#### Backend State
Maintain:
- `state.registry` — loaded once from parquet
- `state.tiles_cache` — tile metadata cache
- `state.preview_cache` — optional in-memory preview cache

---

### 3. Leaflet Frontend Behavior

- Map initializes once.
- On `map.moveend`:
  - Debounce (250–400ms).
  - Call backend: `/api/tiles?...` with current bounds.
- Render:
  - Sidebar list of tiles
  - Footprint rectangles on map
- On selecting a tile:
  - Enable “Visualize Selected”
  - Call `/api/tile/<tile_id>/preview`
  - Add overlay + popup with download link
- Keep:
  - Year filter
  - Status badge
  - Log console (if present in base)

---

### 4. Streaming / AWS Readiness

- Centralize remote fetch in Python:
  ```python
  fetch_bytes(url, use_proxy=True)
