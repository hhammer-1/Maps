# 📍 Location Map

An interactive, self-contained web map for visualizing locations across the US. Built as a single HTML file — no server, no build step, no dependencies to install.

**Live site:** `https://hhammer-1.github.io/Maps`

---

## Purpose

This tool was built for internal use to map and explore a network of locations across the United States. It supports filtering, regional analysis, radius-based proximity search, and dynamic data loading from a shared Google Sheet — making it easy to keep the map up to date without touching code.

---

## Features

### Data Loading
- **Sync from Google Sheets** — paste your sheet URL once and pull live data with one click
- **Upload Excel** — load a local `.xlsx` file for offline use (same format as the Google Sheets) or testing without affecting the shared sheet (data is stored locally in browser)

### Map Visualization
- **Color-coded pins** by category, with a fully customizable legend
- **Shape-coded pins** by category — pick from teardrop, circle, square, diamond, triangle, star, or hexagon per category, so facility types are distinguishable by shape as well as color
- **Regional shading** to highlight target regions, active markets, or any geographic segmentation (US states, Canadian provinces, RoW countries)
- **Radius circles** around selected locations — configurable in miles from the topbar
- **Overlapping pins fan out** automatically — locations that share (or nearly share) coordinates spread into a small fixed-pixel cluster so every pin stays visible and clickable at any zoom level

### Filtering & Discovery
- **Category toggles** — show/hide pin categories independently
- **Tag filters** — multi-value attribute filtering (e.g. by technology, certification, or status) with OR / AND mode
- **Metric filter** — filter pins by Company Revenue, Company EBITDA, Location Revenue, or Location EBITDA using a range slider with editable min/max values. Dropdown only shows metrics that have data; hidden entirely if no financial data is present
- **State shading toggles** — show/hide region groups in the legend
- **Click a state** — instantly see all pins within that state's boundary
- **Click a radius circle** — see all pins within range, with distances

### Sidebar & Navigation
- **List view** — browse all locations with full address
- **Context-aware sidebar** — changes based on what you click (state, circle, company)
- **Drill down** — click a pin popup to see all locations for that parent company
- **Back navigation** — Esc returns to the previous view (e.g. state → company → back to state). Lateral navigation (state → state) resets the stack so Esc never accumulates more than one level
- **Rich popups** — each pin popup shows category, tags, website, financial metrics (Company Revenue, Company EBITDA, Location Revenue, Location EBITDA), and notes
- **Hover to bounce** — hovering a sidebar item bounces the corresponding pin on the map
- **Export CSV** — download whatever is currently shown in the sidebar

### Legend
- Collapsible with per-section scrolling (Locations, Regions, Tags)
- Show all / hide all toggle for location categories
- Tag filter mode toggle (OR / AND)

---

## Architecture

### Single-file design
The entire app is one `index.html` file. CSS, HTML, and JavaScript all live together — no npm, no bundler, no server required. Hosted for free on GitHub Pages.

### External libraries (CDN, no install)
| Library | Purpose |
|---|---|
| [Leaflet.js](https://leafletjs.com) | Map rendering, markers, circles, popups |
| [SheetJS](https://sheetjs.com) | Reading `.xlsx` files from Google Sheets or local upload |
| [CartoDB Voyager](https://carto.com/basemaps/) | Map tile imagery |
| [us-atlas](https://github.com/topojson/us-atlas) | US state boundary polygons |

### Data flow
```
Google Sheet (source of truth)
       │
       │  export?format=xlsx
       ▼
   SheetJS (parse in browser)
       │
       ├─► categories, regions, locations → localStorage (cache)
       │
       ▼
   Leaflet (render pins, circles, state fills)
```

Data is cached in `localStorage` so the map loads instantly on revisit. Each user syncs independently — there is no shared real-time database.

### State interactions
State hover and click are detected via a map-level `mousemove`/`click` handler using a **ray-casting point-in-polygon algorithm** against the GeoJSON boundary data. This means state clicks work correctly even for complex shapes (Hawaii islands, Michigan's Upper Peninsula) and don't block clicks on pins or circles above them.

### Layer ordering (bottom to top)
```
Tile layer (base map)
State fills — z-index 350, non-interactive (pointer events pass through)
Radius circles — z-index 450, interactive
Pins / markers — z-index 600, always on top
```

---

## Google Sheet Setup

The sheet must be shared as **"Anyone with the link can view"**. The app downloads the sheet as an XLSX export directly — no need to publish to web separately.

### Tab names (must contain these keywords)
| Tab | Keyword | Contents |
|---|---|---|
| `Data` | "data" | All location pin data |
| `Locations` | "locations" | Pin categories, colors, sort order |
| `Regions` | "regions" | Regional shading |

### Column layout — `Data` tab
Columns are read by **position**, not header name. Do not reorder or insert columns without updating the code.

| Col | Field | Notes |
|---|---|---|
| A | Location Name | |
| B | Parent Company | Used for "Show all company locations" |
| C | Address | Full address |
| D | Website | Include `https://` |
| E | Category | Must match exactly a name in the Locations tab |
| F | Lat | Decimal |
| G | Lng | Decimal |
| H | Radius | `1` = draw circle, `0` or blank = none |
| I | Tags | Comma-separated e.g. `IM, IBM`. Blank = N/A |
| J | Company Revenue | Raw number (no symbols). Optional |
| K | Company EBITDA | Raw number. Optional |
| L | Location Revenue | Raw number. Optional |
| M | Location EBITDA | Raw number. Optional |
| N | Notes | Free text. Optional |

### Column layout — `Locations` tab (categories)
| Col | Field | Notes |
|---|---|---|
| A | Category name | |
| B | Hex color | e.g. `#3b82f6` |
| C | Sort order | Integer. Positive = top, negative = bottom, blank = alphabetical |
| D | Shape | One of: `teardrop` (default map pin), `circle`, `square`, `diamond`, `triangle`, `star`, `hexagon`. Blank or unrecognized value falls back to `teardrop` |

### Column layout — `Regions` tab
| Col | Field | Notes |
|---|---|---|
| A | State | Full name or abbreviation |
| B | Hex color | e.g. `#92d050`. Blank = no shading |
| C | Label | Legend group name (optional) |

---

## Updating the Map

1. Edit your Google Sheet
2. Go to the live site and paste your Google Sheet shared link into the top bar
3. Click **🔄 Sync**

To update the app itself, edit `index.html` in the GitHub repo and commit. Changes go live within ~30 seconds via GitHub Pages.

---

## Local Excel Testing

Click **⬆ Excel** in the topbar to upload a `.xlsx` file in the same format as the Google Sheet. This is useful for testing changes locally without affecting the shared sheet. The button turns green when local data is loaded. Click **🔄 Sync** to revert to live data from Google Sheets.
