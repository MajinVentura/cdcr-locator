# GOAL: Adapt this app into a CDCR Adult Institution Locator

This folder is **already a complete, working copy** of the WIDOC (Wisconsin)
Adult Institution Locator — a single-file `index.html` web app that finds the
nearest state prisons to a typed address, with driving distance, travel time, and
routes on a map. Your job is to **re-skin it for California's CDCR**, not to
rebuild it. Keep all the interaction logic; change the data, the security scheme,
and the copy.

Read `CLAUDE.md` first — it documents how the app works, the `FACILITIES` data
shape, the external dependencies, and the conventions. Then open `index.html` to
see the working Wisconsin version as your reference implementation.

## What to change

### 1. Facility data (`const FACILITIES = [...]` near the top of `<script>`)
- **The authoritative roster is `FACILITIES-LIST.txt`** in this folder: 31 active
  CDCR adult institutions (as of January 2026) with names and addresses. Use
  **exactly these 31** — do not add or drop any, and do not second-guess the list.
- Replace every Wisconsin entry. Field shape: `name, county, addr, lat, lon,
  security, gender, yearOpened, population, description, details, aerial (Google
  Maps 3D link), url (official cdcr.ca.gov page), image`. **Drop the `phone`
  field** — it is not wanted for this project, so also remove the click-to-call
  phone line from the marker popup template in `index.html`.
- The addresses come from the roster; you still need to research and fill the rest
  per facility: `lat`/`lon` (must be accurate — markers and routing depend on
  them), `security` (Level(s) — see §2), `gender`, `yearOpened`,
  `population`/capacity, `description`, `details`, `url`, `aerial`.
- Source facts from official CDCR pages + Wikipedia. Flag anything you can't
  verify instead of inventing it. Convert relative dates to absolute.

### 2. Security classification scheme (the main real adaptation)
- CDCR uses **Security Levels I–IV** (I = lowest, IV = highest) plus **Reception
  Centers** and camps, and **many institutions house multiple levels at once.**
  Pick and document a single-marker rule (suggested: the **highest level
  present**, or the facility's primary mission).
- Encode security by **both marker shape AND color** plus the small building
  glyph, the way the WI version does (see `securityOf()` and `facilityIcon()`).
  Propose a clean mapping, e.g. a green→red ramp across Level I→IV with a distinct
  shape, plus a slate diamond for Reception/Other. Keep this palette visually
  distinct from the drive-time palette (green/yellow/orange/red).
- Update the security **filter** checkboxes and the **legend** to match.

### 3. Photos
- The 7 JPGs in `images/` are Wisconsin facilities — **remove them** and add
  California ones. Optimize to ~640px-wide JPGs (~50–90KB) with
  `sips -Z 640 -s format jpeg -s formatOptions 68 in.png --out images/<slug>.jpg`.
  Missing images already fall back to a colored header, so partial coverage is OK.

### 4. Copy, theming, map defaults
- Retitle to "CDCR Adult Institution Locator"; update header, intro text, and the
  disclaimer to reference CDCR / California.
- Scope Nominatim geocoding to California/US (replace ", Wisconsin" suffix).
- Recenter the default map view on California (~`[37.2, -119.5]`, zoom ~6) and
  refit any state-specific bounds.
- Update `CLAUDE.md` to describe the CDCR version.

## Keep all of these features exactly as they work today
Address search + debounced autocomplete (keyboard nav); nearest-N with OSRM drive
times; adjustable result count (3/5/10) and sort (drive time / distance) off cached
routes; drive-time color scoring on routes + time-bubbles + result cards; white
route casing; pulsing origin pin; security + gender filters with a "X of N shown"
count; result cards; Browse-all A–Z list; Recent searches (localStorage, last 5);
Reset button; rich popups (photo, stats, official page, Aerial 3D, Open in Maps —
no phone/click-to-call); street/satellite base layers; zoom-gated facility name
labels.

## Constraints (unchanged)
Single self-contained `index.html` (vanilla JS, no build/framework/bundler) plus a
sibling `images/` folder. All deps via CDN/free APIs: Leaflet 1.9.4, OSM tiles,
Esri imagery, Nominatim, OSRM demo server (rate-limited/best-effort — note that
heavy use should move to a keyed/self-hosted endpoint). Use the `:root` CSS custom
properties; don't hard-code colors.

## Before finishing
- Verify the inline JS parses and the app loads with no console errors.
- Spot-check a few `lat`/`lon` values by confirming the marker lands on the prison.
- Commit in small described chunks. (Set up the GitHub remote first — see the
  parent project's two-machine sync workflow.)
