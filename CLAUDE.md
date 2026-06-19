# CDCR Adult Institution Locator

A single-file web app that helps a user find the nearest California Department of
Corrections and Rehabilitation (CDCR) adult institutions to a given address, with
driving distance, travel time, and route lines drawn on a map.

## How it works

Almost everything lives in `index.html` — markup, CSS, and JavaScript in one
file. The only sibling asset is the `images/` folder (local facility photos, if
any). There is **no build step and no package manager**. To run it, open
`index.html` in a browser (or serve the folder, e.g. `python3 -m http.server`).

Flow:
1. User types an address; it is geocoded via the **Nominatim** API
   (`nominatim.openstreetmap.org`), scoped to California / US.
2. Distances to every facility are computed (haversine for ranking).
3. Nearest facilities get real driving routes/times from the **OSRM** API
   (`router.project-osrm.org`) and route lines are drawn.
4. Results render in a side panel; markers + routes render on a **Leaflet** map
   centered on California (`[37.2, -119.5]`, zoom 6).

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
one object per institution with `name`, `county`, `addr`, `lat`, `lon`,
`security`, `mission`, `gender`, `yearOpened`, `population`, `description`,
`details`, optional `aerial` (Google Maps 3D link), `url` (official cdcr.ca.gov
page), and `image`. There are **31 active adult institutions** (as of January
2026). To add or correct an institution, edit this array — keep the field shape
consistent and supply accurate `lat`/`lon` (markers and routing depend on them).
There is **no `phone` field** (deliberately dropped for this project).

`image` is either a remote URL (several are Wikimedia) or a relative path into the
local `images/` folder (e.g. `images/<slug>.jpg`). Local photos are optimized to
~640px-wide JPGs (use `sips -Z 640 -s format jpeg -s formatOptions 68 in.png --out
images/<slug>.jpg`). A broken/`null` image falls back to a plain blue header, so
partial photo coverage is fine.

## Security levels & mission/type (the CDCR-specific model)

CDCR uses **Security Levels I–IV** (I = lowest, IV = highest) plus **Reception
Center** designations, and **most institutions house multiple levels at once.**

- `security` records the **full set** of levels each facility houses, listed not
  collapsed (e.g. `"Levels I–IV"`, `"Levels I, III, IV"`, `"Reception Center;
  Levels I–III"`). `secCategories(s)` parses this string into a canonical array
  of `I / II / III / IV / Reception` — it handles ranges (`I–IV`), explicit lists
  (`I, III, IV`), single levels (`Level II`), and "Reception". The security
  filter matches a facility if it houses ANY checked level; `facilityPasses()`
  uses this.
- `mission` is the facility's primary role/type. `missionOf(f)` maps the
  `mission` string to one of seven keys: `gp` General Population, `rc` Reception
  Center, `wm` Women's, `md` Medical / Health Care, `sa` Substance-Abuse
  Treatment, `cv` Conservation / Training, `rh` Rehabilitation / Reentry.

### Marker encoding
Markers are a consistent round **badge**: **fill color = security intensity**
(`securityOf()` returns the color of the *highest* level housed — green I → blue
II → orange III → red IV, slate for Reception-only), and a white **glyph = mission**
(`MISSION_GLYPH`). This shows both dimensions without lying about a single
"level." The legend swatches are generated from `SEC_COLORS` / `MISSION_GLYPH` so
they stay in sync. Result-facility markers are drawn enlarged; the entered-address
pin is a purple pulsing teardrop. The drive-time palette (routes, bubbles, result
cards) is intentionally kept distinct from the security palette.

## Conventions

- Keep logic/markup/styles in the single self-contained `index.html` unless
  there's a strong reason to split (it's meant to be trivially portable /
  openable offline-ish). Local facility photos in `images/` are the one
  exception — they travel with the file and load via relative path.
- Match the existing vanilla-JS style — no framework, no bundler.
- CSS custom properties (`--bg`, `--accent`, etc.) live in `:root`; reuse them
  rather than hard-coding colors.
- Source facts from official CDCR pages + Wikipedia; flag anything unverifiable
  rather than inventing it, and convert relative dates to absolute.

## Working on two machines (office + home)

This repo is synced via GitHub. Before switching machines: `git push`. After
arriving: `git pull`. Commit in small, described chunks so history is useful.
