# CLAUDE.md — AI Assistant Guide for Searchable-PCI

## Project Overview

Searchable-PCI is a single-file web application that provides a search engine for NCDR CathPCI Registry coding guidance. It is built by Carta Healthcare and deployed to GitHub Pages. The application contains ~610 embedded searchable items across 5 data sources related to medical coding, including synonym-aware search for medication brand/generic names and coronary artery vessel names.

## Repository Structure

```
/
├── CLAUDE.md              # This file
├── README.md              # Brief project description
├── files.zip              # Archive of the latest application files (~109KB)
├── index.html             # Deployed application (6121 lines, ~754KB)
├── .github/
│   └── workflows/
│       └── deploy.yml     # GitHub Actions workflow for GitHub Pages
└── site/                  # Extracted copy of files.zip (canonical latest version)
    ├── index.html         # Latest application (6151 lines) — most up-to-date
    ├── deploy.yml         # Workflow copy
    └── README.md          # Detailed project documentation
```

### Important Version Note

There is currently a version discrepancy:
- **`site/index.html`** (6151 lines) is the most up-to-date version and matches `files.zip`. It includes coronary artery aliases in the synonym search.
- **Root `index.html`** (6121 lines) is the version deployed by GitHub Pages (it is served from the repo root). It is slightly behind `site/index.html` — it includes medication synonyms but not coronary artery aliases.

**The zip / `site/index.html` is the source of truth** for feature completeness. When making changes, update `site/index.html`, then update `files.zip`, then sync the root `index.html`.

## Technology Stack

- **Languages**: HTML5, CSS3, Vanilla JavaScript (ES6+)
- **Framework**: None — zero dependencies, no build tools, no package manager
- **Fonts**: Space Grotesk (headings), system fonts (body)
- **Deployment**: GitHub Pages via GitHub Actions
- **Server**: None required — fully client-side, works offline

## Architecture

### Single-File Design

Everything is contained in `index.html`. Using `site/index.html` as the canonical reference (~6,151 lines):

| Section | Lines (approx) | Purpose |
|---------|----------------|---------|
| CSS | 8–527 | All styles, responsive design, Carta branding, NCDR Resources tables |
| HTML | 529–566 | Page structure: header (search only), filters, results |
| JavaScript | 567–6151 | Search logic, synonym lookup tables, and all embedded data |

### Embedded Data Structure

All data is stored in a JavaScript constant `embeddedData` with five arrays:

```javascript
const embeddedData = {
  mainDefinitions: [],            // ~346 items — Data Dictionary entries
  supportingDefinitions: [],      // ~95 items  — Supporting Definition entries
  additionalCodingDirectives: [], // ~145 items — Q&A format coding directives
  pendingUpdates: [],             // ~22 items  — Pending Definition Updates
  ncdResources: []                // 2 items    — NCDR Resource references
}
```

Each item has a `type` field and a pre-computed lowercase `searchText` field used for search matching.

### Key JavaScript Functions

| Function | Purpose |
|----------|---------|
| `loadData()` | Initializes data from `embeddedData` into `allData` |
| `escapeRegex(str)` | Escapes special regex characters in a string |
| `getExpandedSearchTerms(searchTerm)` | Expands a search term with medication/vessel synonyms |
| `searchData(searchTerm)` | Full-text search using `String.includes()` across expanded terms |
| `performSearch()` | Main search handler, triggered on input events |
| `displayResults(results)` | Renders search results to DOM; shows synonym expansion info |
| `renderResult(item)` | Creates HTML for a single result card |
| `highlightText(text, searchTerm)` | Highlights matching terms (and synonym terms) in results |
| `cleanText(text)` | Formats and sanitizes display text; handles CROSSWALK sections |
| `getTypeBadgeClass(type)` | Maps data source type to CSS badge class |

Note: `matchesFilter(type)` and `matchesSearch(item)` are local helper functions defined inside `searchData()`, not standalone functions.

### Synonym Lookup System

The application supports synonym-aware search for two categories:

**Medication synonyms** (`medicationSynonyms` array):
Groups equivalent brand names, generic names, and abbreviations for cardiac medications (calcium channel blockers, statins, antiplatelet agents, anticoagulants, GP IIb/IIIa inhibitors, beta blockers, ACE inhibitors, ARBs, thrombolytics, nitrates, antiarrhythmics, diuretics, vasopressors, and other cardiac medications).

Examples:
- `"plavix"` → also searches `"clopidogrel"`
- `"asa"` → also searches `"aspirin"`, `"acetylsalicylic acid"`
- `"lopressor"` → also searches `"toprol"`, `"metoprolol"`

**Coronary artery aliases** (`coronaryArteryAliases` array, currently in `site/index.html` only):
Groups abbreviations with full anatomical names for coronary vessels and segments.

Examples:
- `"lad"` → also searches `"left anterior descending"`
- `"rca"` → also searches `"right coronary artery"`
- `"lcx"` → also searches `"left circumflex"`, `"circumflex"`
- `"prca"` → also searches `"proximal right coronary artery"`

Both arrays are merged into a single `synonymLookup` dictionary for O(1) lookup.

### Global State

- `allData` — All loaded items (object with arrays by type)
- `currentFilter` — Active filter type (string, default `'all'`)
- `currentSearchTerm` — Current raw search query (string)
- `currentExpandedTerms` — Array of expanded search terms after synonym expansion

## NCDR Resources Data Type

The `ncdResources` array holds structured reference resources, not text-based coding guidance. Currently two items:

**1. Qualitative/Quantitative Crosswalk**
```javascript
{
  title: "Qualitative/Quantitative Crosswalk",
  type: "NCDR Resources",
  isImage: true,
  crosswalkData: [{ term, value, group }],  // group: "lt50" | "gte50" | "gte70" | "gte90" | "100"
  searchText: "..."
}
```

**2. Coronary Artery Segment Diagram**
```javascript
{
  title: "Coronary Artery Segment Diagram",
  type: "NCDR Resources",
  isImage: true,
  segmentDiagram: true,
  legendGroups: [{ label, color }],
  segmentData: [{ num, name, abbrev, group }],  // group: "lm" | "proxlad" | "middistlad" | "ramus" | "rca" | "circ" | "pda"
  searchText: "..."
}
```

NCDR Resources items use `title` (not `elementName`) and may omit `seqNum`. The `renderResult()` function handles these specially, rendering tables instead of text.

## CSS Conventions

- **Color scheme**: Carta Healthcare branding with CSS custom properties (`--carta-teal-dark`, `--carta-teal`, `--carta-blue`, `--carta-gray-*`, `--carta-accent`)
- **Naming**: BEM-like convention (`.result-item`, `.result-header`, `.type-badge`)
- **Type badges**: `.type-dictionary`, `.type-supporting`, `.type-acd`, `.type-pending`, `.type-ncdr-resources`
- **Layout**: Flexbox-based responsive design

### NCDR Resources CSS Classes

| Class | Purpose |
|-------|---------|
| `.crosswalk-table` | Styles the qualitative/quantitative crosswalk table |
| `.crosswalk-group-lt50` | Row color for <50% stenosis entries |
| `.crosswalk-group-gte50` | Row color for ≥50% stenosis entries |
| `.crosswalk-group-gte70` | Row color for ≥70% stenosis entries |
| `.crosswalk-group-gte90` | Row color for ≥90% stenosis entries |
| `.crosswalk-group-100` | Row color for 100% (occlusion) entries |
| `.segment-table` | Styles the coronary artery segment table |
| `.segment-legend` | Container for the color legend |
| `.segment-legend-item` | Individual legend entry (swatch + label) |
| `.segment-legend-swatch` | Colored square in the legend |
| `.segment-group-lm` | Row color for Left Main segments |
| `.segment-group-proxlad` | Row color for Proximal LAD segments |
| `.segment-group-middistlad` | Row color for Mid/Distal LAD segments |
| `.segment-group-ramus` | Row color for Ramus segments |
| `.segment-group-rca` | Row color for RCA segments |
| `.segment-group-circ` | Row color for Circumflex segments |
| `.segment-group-pda` | Row color for PDA segments |
| `.segment-abbrev-cell` | Styled abbreviation cell |
| `.segment-num-cell` | Styled segment number cell |
| `.segment-source` | Italic attribution footer |

## Page Structure

The header contains **only** the search input (no title or logo). The page title appears in the `#intro` div, which is shown when no search is active and hidden when results are displayed:

```html
<div class="header">           <!-- Sticky header: search input only -->
<div id="filters">             <!-- Filter buttons row -->
<div id="resultsInfo">         <!-- "Found N results..." text + synonym info -->
<div id="intro">               <!-- Title + description, hidden during search -->
<div id="results">             <!-- Result cards -->
```

Note: The Carta Healthcare logo/branding bar was removed in an earlier update.

## Development Workflow

### Local Development

No build step. Open `index.html` directly in a browser:

```bash
# Open the root-level index.html
open index.html           # macOS
xdg-open index.html       # Linux

# Or to use the most up-to-date version:
open site/index.html
```

### Deployment

The GitHub Actions workflow (`.github/workflows/deploy.yml`) deploys **the entire repository root** as a GitHub Pages artifact. The root-level `index.html` is served at the site URL. Deployment triggers automatically on push to `main`, or manually via `workflow_dispatch`.

```
push to main → GitHub Actions → uploads repo root (path: '.') → serves root index.html
```

### Updating Data

To update searchable data:
1. Modify the `embeddedData` object inside `site/index.html`
2. Ensure each new item has a `searchText` field (lowercase concatenation of all searchable fields)
3. Update `files.zip` with the modified `site/index.html`
4. Sync the changes to the root `index.html` to keep the deployed version current
5. Commit and push to `main`

### Adding Medication Synonyms

Add new synonym groups to the `medicationSynonyms` array (arrays of equivalent names):
```javascript
["brandName", "genericName", "abbreviation"],
```
The `synonymLookup` is built automatically from all groups at page load.

### Adding Coronary Artery Aliases (site/index.html)

Add new alias groups to the `coronaryArteryAliases` array:
```javascript
["abbreviation", "full anatomical name"],
```

## Build & Test Commands

There are **no build or test commands**. This project has:
- No `package.json`
- No test framework
- No linter configuration
- No CI checks beyond deployment

Validation is manual — open `index.html` in a browser and verify search functionality.

## Key Conventions for AI Assistants

1. **Single-file architecture** — Do not split `index.html` into separate files without explicit instruction. The monolithic design is intentional for portability and deployment simplicity.

2. **No dependencies** — Do not introduce npm packages, CDN imports, or build tools unless explicitly requested. The zero-dependency design is a feature.

3. **Data lives in the HTML** — All ~610 items are embedded directly in JavaScript. There is no external API, database, or JSON file.

4. **The zip / `site/` is the source of truth** — The most complete deployable files live in `files.zip` / `site/index.html`. The root `index.html` must be kept in sync when changes are made.

5. **Branding matters** — Carta Healthcare branding (colors, fonts) must be preserved in any UI changes. The logo bar has been intentionally removed; do not re-add it without explicit instruction.

6. **Search is client-side** — All search happens in-browser. There is no server-side component.

7. **Synonym expansion is part of search** — The search system expands queries using `getExpandedSearchTerms()`. When modifying search behavior, preserve synonym lookup compatibility.

8. **GitHub Pages deployment** — The workflow deploys from the repo root (`path: '.'`). The root-level `index.html` is what users see at the live URL.

9. **NCDR Resources are structured data** — Unlike the text-based coding guidance items, NCDR Resources have custom structured fields and are rendered as tables. Treat them differently from the other four data types.

## Data Source Types

| Type | Badge Class | Count (approx) | Fields |
|------|-------------|----------------|--------|
| Data Dictionary | `.type-dictionary` | 346 | seqNum, elementName, codingInstructions, targetValue |
| Supporting Definition | `.type-supporting` | 95 | seqNum, elementName, supportingDefinition |
| Additional Coding Directives | `.type-acd` | 145 | seqNum, elementName, question, answer, postedDate |
| Pending Definition Updates | `.type-pending` | 22 | seqNum, elementName, codingInstruction, notes, targetValue |
| NCDR Resources | `.type-ncdr-resources` | 2 | title, isImage, crosswalkData OR segmentDiagram+segmentData+legendGroups |

## Common Tasks

### Adding a new data entry
Add an object to the appropriate array in `embeddedData` with all required fields plus a `searchText` field containing a lowercase concatenation of all searchable text.

### Adding a new data source type
1. Add a new array to `embeddedData`
2. Add a new filter button in the HTML (`<button class="filter-btn" data-filter="...">`)
3. Add CSS for the new type badge (`.type-<name>`)
4. Update `getTypeBadgeClass()` to map the new type
5. Update `renderResult()` to display fields for the new type
6. Add the new array to `searchData()`'s forEach loops

### Modifying search behavior
The `searchData()` function performs substring matching on the `searchText` field across all expanded terms returned by `getExpandedSearchTerms()`. To change search logic (e.g., fuzzy matching, ranking), modify this function. To change synonym expansion logic, modify `getExpandedSearchTerms()`.

### Adding a new synonym group
- For medications: append to the `medicationSynonyms` array
- For coronary vessels: append to the `coronaryArteryAliases` array (in `site/index.html`)
- Both are merged into `synonymLookup` at build time — no other changes needed
