# Filament Stock — Project Context (`GEMINI.md`)

## Overview
- **App:** `index.html` — single-file, mobile- and desktop-friendly 3D printer filament inventory web app for Yanick Therien (+ 2 friends).
- **Stack:** Self-contained HTML + Tailwind CSS (CDN) + vanilla JavaScript. No build step, no npm packages.
- **Printers & Slicer:** FlashForge Creator 5 Pro (`256×256×256 mm`, 4 toolheads with hardened steel nozzles, `+5°C` nozzle offset) & Adventurer 5M; OrcaSlicer `2.5.0.0`.
- **Hosting:** GitHub repo `filament-stock` (published via GitHub Pages / embedded in Google Sites).

## Live Backend & Links
- **Master Google Sheet (`3D Filament Inventory – Yanick`):** `https://docs.google.com/spreadsheets/d/14I7jNPzWpfI1Lu_LsKwZPrc6ui-elh4Otef3I1rJYJQ/edit`
- **Apps Script Web App Endpoint (`DEFAULT_API_URL`):** `https://script.google.com/macros/s/AKfycbxh_nygVHZnxTocHxeqAjiTc3w9k3HxRMpF1LVluiU24oFLoEd2Dte8ebL4838Q5haeMA/exec`
- **Google Drive OrcaSlicer Sync Folder:** `OrcaSlicer_Filaments_Sync` (stores generated OrcaSlicer 2.5 `.json` filament profiles).

## Google Sheet Schema (`In Stock` tab — Row 5 Headers)
`ID` | `Brand` | `Material` | `Color` | `Color Hex` | `Diameter` | `Status` | `Printer / Toolhead` | `Notes` | `Drying Info` | `Humidity / Storage` | `Print Temps` | `Particularities / Tips` | `Date Added` | `Last Updated` | `Orca JSON` | `Orca JSON URL` | `Max Flow (mm³/s)` | `Fan % (Min/Max)` | `Swatch_URL` | `Swatch_Status`

- **Exact Status Mapping (Sheet ↔ App):**
  - `In Stock` ↔ `in_stock`
  - `Ordered - Not Received` ↔ `ordered`
  - `Low` ↔ `low`
  - `Empty` ↔ `empty` (also appends a row to the `Reorder & Finished Log` tab)
- **One row per physical spool** (`FIL-001`, `FIL-002`, ...).

## Apps Script API (`Code.gs`)
- `GET` → returns `{ ok: true, spools: [...] }` reading all columns above.
- `POST` (`Content-Type: text/plain`, JSON body) supports:
  - `{ action: "upsert", spool: {...} }`
  - `{ action: "updateStatus", id, status }`
  - `{ action: "delete", id }`
  - `{ action: "syncOrcaToDrive", spoolId?, force? }` (0-token local generator; no Gemini API key needed)
  - `{ action: "syncSwatches", spoolId?, force? }` (checks `3dfilamentprofiles.com` first, then `filamentcolors.xyz` API fallback)

## UI & Feature Rules
- Keep cards clean and uncluttered: put technical details (Humidity, Drying, Print Temps, Max Flow / Fan %, OrcaSlicer 2.5 JSON link, Tips, and individual **Sync JSON** button) inside the `i` info modal.
- **Smart Swatch Palette Icon (in `i` modal header):**
  - **Green (`matched`):** Direct link on `3dfilamentprofiles.com`.
  - **Teal (`fallback`):** Found on `filamentcolors.xyz`.
  - **Amber (`missing`):** Copies `Brand Material Color (Hex)` to clipboard and opens `https://3dfilamentprofiles.com/filaments` to submit.
- **Moisture Droplet Icon:** Critical (red) for ASA, PC+, HIPS, PETG-CF, TPU; Moderate (amber) for PETG, ABS; none for PLA.

## Editing Guidelines for AI Agents
- Always make **small, targeted edits** to `index.html`—never rewrite the whole file unnecessarily.
- Never remove or alter `DEFAULT_API_URL`, `localStorage` try/catch wrappers, or existing Sheet sync/filter/modal features unless explicitly asked.
- Never use `alert()`, `confirm()`, or `prompt()`—use the built-in modals and `showToast()`.
- Keep everything in `index.html` self-contained so it works identically on GitHub Pages, Google Sites, and local preview.
