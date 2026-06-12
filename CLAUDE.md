# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-file offline HTML application (`index.html`) that teaches semiconductor chip cost economics to product managers. Chinese-language UI with English technical terms. No build tools, no frameworks, no backend — open the file directly in a browser.

## How to develop

```bash
# Preview — just open the file
open index.html

# No install, build, lint, or test steps. This is a zero-dependency static HTML file.
```

## Architecture

**Data layer** — JS constants at the top of the `<script>` block (~line 216+):
- `DOMAINS` — L2 five cost domains (D1–D5: Silicon Die, Package, Test, NRE Amortization, IP Royalty)
- `COST_ITEMS` — L3 ~30 cost items, each keyed to a `domainId`
- `SUPPLIERS` — L4 ~100 supplier entries across layers (IP, EDA, Foundry, Mask, Wafer, OSAT, Substrate, ATE, Memory, Equipment, Fabless)
- `TARGET_CHIPS` — comparison chip presets (RK3566, RK3588, RK3568, etc.) with parameters for the calculator
- `WAFER_PRICES` — price matrix by process node × foundry
- `PACKAGE_FORMATS` — reference library of packaging types
- `GLOSSARY` — term definitions

**State** — `STATE` object (~line 667) tracks current tab, selected chip, filters, and calculator state.

**Persistence** — edits to data arrays are saved to `localStorage` under key `chip-cost-edits-v1`. `applyEdits()` merges saved changes on load. `saveAll()` serializes all data arrays.

**Rendering** — tab-based single-page navigation. Each tab (`overview`, `domains`, `items`, `suppliers`, `chips`, `packages`, `wafer`, `calc`, `formulas`, `glossary`) has a dedicated `render*()` function that writes innerHTML into `<main>`.

**UI components:**
- Side drawer (`#drawer`) for detail drill-downs on cost items and suppliers
- Modal (`#editModal`) for CRUD operations on all data entities
- Backdrop overlay for both
- Global search builds an index across all data types and renders grouped results
- Theme toggle switches between light/dark via CSS custom properties on `<body>`

**Cost calculation formulas** (driven by `TARGET_CHIPS` parameters):
- DPW ≈ π·(D/2)²/S − π·D/√(2·S)
- Yield (Murphy model): Y = ((1 − exp(−A·D0))/(A·D0))²
- Die cost = wafer price / (DPW × Y)
- Unit cost = die + package + test + IP royalty + NRE amortization + overhead %

**CSV files** — `chips.csv` and `suppliers.csv` are external data sources; the app's JS constants are the primary data store.

**架构设计初稿.md** — architecture design document (Chinese) describing the pyramid structure, content model, page layout, and implementation plan. Reference for understanding design intent.

## Key patterns

- All data is mutable at runtime via the edit modal; changes persist to localStorage
- Search index is rebuilt on every keystroke from the live data arrays
- The calculator form is dynamically generated from `TARGET_CHIPS` keys
- Chip presets are selectable via the top-bar picker; the active chip drives all cost calculations and domain percentage displays
- Supplier tiles and cost item rows are clickable, opening a detail drawer with related entities cross-linked
