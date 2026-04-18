# PK Props Vessels Catalog — Project Summary

**Last updated:** April 10, 2026

---

## What It Is

An interactive HTML catalog for PK Props (PhotoKitchen), a food photography and videography company. The catalog is an **internal team tool** — it gives the whole team a quick overview of all the props they currently own, so they can efficiently select vessels for pre-production decks when planning shoots. The team accesses it via a **GitHub Pages link** so everyone can open the latest version from anywhere.

---

## File Structure

The catalog is split across multiple files for performance and maintainability:

- **`index.html`** — Shell file. Contains all CSS, UI logic, and JS. No vessel data or images. Hosted on GitHub Pages.
- **`data/[color].js`** — One JS file per color (e.g. `red.js`, `white.js`, `green.js`). Each file contains that color's `vessels` array, `IMAGES` object, `PDF_PAGES` object, and `VESSEL_PAGE` mapping. Loaded dynamically when a color is selected.

**Repository:** `https://github.com/photokitchenfood/vessels`

---

## Architecture

Built entirely in HTML + vanilla JavaScript. Key systems:

- **Vessel data** — flat JS array of vessel objects per color file, each with `id`, `color`, `category`, `name`, and optional dimension fields (`diameter`, `length`, `width`, `height`, `qty`)
- **Image system** — `IMAGES` object keyed by vessel ID, storing base64-encoded photos. `PDF_PAGES` stores slide placeholder images keyed by page name. `VESSEL_PAGE` maps each vessel ID to its corresponding slide page key.
- **Filtering engine** — real-time filtering by color, category, size, dimension range, and quantity
- **Lazy loading** — `IntersectionObserver` for performance as catalog scales
- **Color theming** — selected color dynamically sets `--accent` and `--accent-pale` CSS variables, influencing multiple UI states
- **Save button** — appears only when changes are made. Detects which color(s) were edited and downloads each as a dated JS file (e.g. `260306-green.js`). Snapshots dirty colors before switching away so multiple colors can be saved in one go.

---

## ID Convention

`[color prefix]-[category prefix]-[number]`

**Color prefixes:** `r` (Red), `w` (White), `b` (Blue), `g` (Green), and others planned

**Category prefixes:**

| Prefix | Category |
|---|---|
| `sp` | Serving Plates |
| `p` | Plates |
| `b` | Bowls |
| `r` | Ramekins |
| `bt` | Baskets & Trays |
| `cg` | Cups & Glasses |
| `jb` | Jars & Bottles |
| `pv` | Pitchers & Vases |
| `pp` | Pots & Pans |
| `bs` | Boards & Stands |
| `lb` | Lunchbox |
| `ot` | Others |

New vessels always go at the bottom of their category to preserve ID chronology. When relocating a vessel between categories, it gets a new ID at the bottom of the destination category.

---

## Vessel Descriptions

Follow this format: **[Type], [Finish] [Material], [Shape/Body], [Key Details], [Style Notes]**

Descriptions should be visually specific enough to be useful as search terms — include texture, glaze finish, silhouette, notable features, and aesthetic notes. Example:

> *Bowl, Matte Speckled Stoneware, Wide Shallow Form, Irregular Rim, Rustic Artisan Feel*

Avoid vague terms. Prefer: *pinched spout, tapered cylinder, waffle texture, warm cream with terracotta flecks* over: *nice bowl, white ceramic.*

---

## Image Workflow

1. Upload vessel photos in exact sequence order matching the catalog slides (left-to-right, top-to-bottom)
2. Compress: `quality=90`, `max_width=800`
3. Encode as base64
4. Insert into `IMAGES` object in the relevant `data/[color].js` file, keyed by vessel ID
5. Map to vessel IDs in chronological sequence order

Slide placeholder images (from Google Slides PNGs) are embedded in `PDF_PAGES` and mapped via `VESSEL_PAGE` — these show automatically until a real photo is added for each vessel.

---

## Broken Vessels & Photo Handling Rules

<!-- ✅ NEW SECTION -->

- **Never skip or remove broken vessels.** Items marked with an X on the Google Slides placeholder are kept in the catalog as historical records. They receive a black "X" badge in the UI and must not be omitted or renumbered around.
- **Never skip any item when embedding photos.** If a photo appears to be missing for a vessel in a batch, always ask before proceeding — the catalog is structured to include all vessels, including broken ones.
- **Broken vessels get photos when available.** If a photo exists for a broken vessel, it should be assigned like any other — skip only if no photo was actually provided for it.

---

## Session Workflow

**Starting a new session:**
1. Upload `index.html` + the relevant `data/[color].js` file(s)
2. Claude makes changes and outputs updated file(s)
3. Download and replace in your local repo
4. Commit and push via GitHub Desktop — GitHub Pages updates automatically

**Adding a new color from scratch:**
1. Upload the Google Slides pages as PNGs (up to 20 per message)
2. Claude reads measurements and builds `data/[color].js` with all placeholders and slide images embedded
3. Claude also outputs a self-contained **`[color]-preview.html`** — open this locally to verify cards, names, dimensions, and category chips before touching the repo
4. Once preview looks good, download `data/[color].js`, add to repo, push

**Photo embedding (per batch):**
1. Upload `index.html` + the current `data/[color].js`
2. Send photos in sequence order matching the catalog slides (left-to-right, top-to-bottom)
3. Claude compresses (`quality=90`, `max_width=800`), encodes as base64, and embeds into the `IMAGES` object
4. Claude outputs both an updated `data/[color].js` **and** a `[color]-preview.html`
5. Open the preview HTML locally to verify photos and names look correct
6. Once satisfied, replace `data/[color].js` in your local repo, commit and push via GitHub Desktop

**Saving edits from the live site:**
1. Make changes on the live GitHub Pages site (edit names, dimensions, add vessels)
2. Click the Save button — downloads `YYMMDD-[color].js` for each modified color
3. Replace the relevant files in `data/`, commit and push

**Files to upload per session:**
- Photo embedding: `index.html` + `data/[color].js`
- New color setup: Google Slides PNGs + `index.html`
- Preview check only: no upload needed — Claude outputs preview from current working files

---

## Current Status

| Color | Status |
|---|---|
| 🔴 Red | ✅ Complete (121 vessels) |
| ⚪ White | ✅ Complete |
| 🔵 Blue | 🔄 In progress (b-pv-1 done) |
| 🟢 Green | ✅ Complete (188 vessels, all photos embedded) |
| All others | ⏳ Pending |

---

## UI Layout

### Desktop
- **Header** (sticky, 64px) — "PhotoKitchen" logo + "Props Catalog 2.0 — Vessels" meta + Save button (hidden until changes made)
- **Sidebar** (260px, sticky) — Color dropdown → Category chips → Size chips → Dimension Range → Quantity chips
- **Main grid** — responsive card grid with lazy-loaded images

### Mobile
- Header → Sidebar filters → Card grid

---

## Color Dropdown Behavior

- Single-select (one color at a time). Rendering all colors simultaneously caused visual overload during shoot planning.
- Shows **"Select a Color"** as the initial prompt; cards only render once a color is selected
- On selection: dropdown adopts card chip styling — tinted background (`swatch + 1A`), matching border, matching text color, matching arrow color (updated dynamically via `backgroundImage`)
- White special case: white background, gray (`#999`) text and arrow
- Color dot indicator (12px) appears inside the dropdown, 15px from left edge
- Resets cleanly via "Reset All Filters"

---

## Typography & Branding

| Element | Font | Notes |
|---|---|---|
| Logo | Barlow Semi Condensed | |
| Color dropdown | Syne 21px | Larger to compensate for smaller x-height vs DM Mono |
| Card titles | Syne | |
| Search input, tags, metadata | DM Mono | |

---

## Key Technical Notes

- Use `backgroundColor` (not `background` shorthand) when also setting `backgroundImage` via JS — the shorthand resets `background-image` and causes conflicts
- JS string literals in vessel names must not contain unescaped single quotes — breaks the data array
- File size monitoring is essential — compression at `quality=90`, `max_width=800` keeps the best balance between visual quality and file size
- `loadColorDataWithSnapshot()` must be used instead of `loadColorData()` on color switch — this snapshots any dirty color data before unloading so it can be saved later
- Cards with missing dimensions are highlighted in yellow (`no-dims` class) as a visual flag for the team
- **`IMAGES` variable must be named exactly `IMAGES =` in every color JS file** — not `GREEN_IMAGES`, `RED_IMAGES`, or any other prefix. index.html reads from the global `IMAGES` object directly; any other name will silently fall back to slide placeholders for all vessels.
- **Preview HTML must match the styling of `index.html`.** Use the same fonts (Syne, DM Mono, Barlow Semi Condensed), background color (`#F5F2EE`), card layout, chip styles, and color accent behavior as the live catalog shell. <!-- ✅ NEW -->
- **Preview HTML must show a vessel count per category chip.** Display the count of vessels in each category chip at the top of the preview so counts can be cross-checked against the individual image files before uploading. <!-- ✅ NEW -->
- **Session image limit is 100 images per conversation.** Plan uploads in batches of 20 (the per-message limit) and track the running count. At 80/100, flag that only one batch remains and advise splitting remaining work across a new session if needed. Always state the current count after each batch (e.g. "40/100 images used this session") so Andi can plan ahead.
