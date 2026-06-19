# WIDOC Adult Institution Locator

A single-file web app that helps a user find the nearest Wisconsin Department of
Corrections (WIDOC) adult institutions to a given address, with driving distance,
travel time, and route lines drawn on a map.

## How it works

Almost everything lives in `index.html` — markup, CSS, and JavaScript in one
file. The only sibling asset is the `images/` folder (local facility photos). There
is **no build step and no package manager**. To run it, open `index.html` in a
browser (or serve the folder, e.g. `python3 -m http.server`).

Flow:
1. User types an address; it is geocoded via the **Nominatim** API
   (`nominatim.openstreetmap.org`), scoped to Wisconsin / US.
2. Distances to every facility are computed (haversine for ranking).
3. Nearest facilities get real driving routes/times from the **OSRM** API
   (`router.project-osrm.org`) and route lines are drawn.
4. Results render in a side panel; markers + routes render on a **Leaflet** map.

## External dependencies (all via CDN / public APIs — internet required)

- **Leaflet 1.9.4** — map library (CSS + JS from unpkg).
- **OpenStreetMap** raster tiles — street base layer.
- **Esri World Imagery** — aerial base layer.
- **Nominatim** — address geocoding.
- **OSRM** (`router.project-osrm.org`) — driving routes & travel time.

Note: Nominatim and the demo OSRM server have usage limits and are best-effort;
heavy use should eventually move to a keyed/self-hosted endpoint.

## The facility data

`const FACILITIES = [...]` near the top of the `<script>` is the source of truth:
one object per institution with `name`, `phone`, `county`, `addr`, `lat`, `lon`,
`security`, `gender`, `yearOpened`, `population`, `description`, `details`,
optional `aerial` (Google Maps 3D link), `url` (official DOC page), and `image`.
To add or correct an institution, edit this array — keep the field shape
consistent and supply accurate `lat`/`lon` (markers and routing depend on them).

`image` is either a remote URL (several are Wikimedia) or a relative path into the
local `images/` folder (e.g. `images/fox-lake.jpg`). Local photos are optimized to
~640px-wide JPGs (use `sips -Z 640 -s format jpeg -s formatOptions 68 in.png --out
images/<slug>.jpg`). A broken/`null` image falls back to a plain blue header.

## Conventions

- Keep logic/markup/styles in the single self-contained `index.html` unless
  there's a strong reason to split (it's meant to be trivially portable /
  openable offline-ish). Local facility photos in `images/` are the one
  exception — they travel with the file and load via relative path.
- Match the existing vanilla-JS style — no framework, no bundler.
- CSS custom properties (`--bg`, `--accent`, etc.) live in `:root`; reuse them
  rather than hard-coding colors.

## Working on two machines (office + home)

This repo is synced via GitHub. Before switching machines: `git push`. After
arriving: `git pull`. Commit in small, described chunks so history is useful.
