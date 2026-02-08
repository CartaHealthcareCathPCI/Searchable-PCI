# CLAUDE.md — AI Assistant Guide for Searchable-PCI

## Project Overview

Searchable-PCI is a single-file web application that provides a search engine for NCDR CathPCI Registry coding guidance. It is built by Carta Healthcare and deployed to GitHub Pages. The application contains 608 embedded searchable items across 4 data sources related to medical coding.

## Repository Structure

```
/
├── README.md          # Brief project description (root level)
├── files.zip          # Archive containing deployable project files
│   ├── index.html     # Complete application (749KB single-file SPA)
│   ├── deploy.yml     # GitHub Actions workflow for GitHub Pages
│   └── README.md      # Detailed project documentation
└── CLAUDE.md          # This file
```

The production application lives inside `files.zip`. The `index.html` inside the zip is the entire application — HTML, CSS, JavaScript, and all data are embedded in a single file.

## Technology Stack

- **Languages**: HTML5, CSS3, Vanilla JavaScript (ES6+)
- **Framework**: None — zero dependencies, no build tools, no package manager
- **Fonts**: Space Grotesk (headings), system fonts (body)
- **Deployment**: GitHub Pages via GitHub Actions
- **Server**: None required — fully client-side, works offline

## Architecture

### Single-File Design

Everything is contained in `index.html` (~5,672 lines):

| Section | Lines (approx) | Purpose |
|---------|----------------|---------|
| CSS | 8–380 | All styles, responsive design, Carta branding |
| HTML | 384–435 | Page structure: header, search, filters, results |
| JavaScript | 440–5672 | Search logic + embedded data (608 items) |

### Embedded Data Structure

All data is stored in a JavaScript object `embeddedData` with four arrays:

```javascript
embeddedData = {
  mainDefinitions: [],          // 346 items — Data Dictionary entries
  supportingDefinitions: [],    // 95 items — Supporting Definition entries
  additionalCodingDirectives: [],// 145 items — Q&A format coding directives
  pendingUpdates: []            // 22 items — Pending Definition Updates
}
```

Each item has a `type` field and a pre-computed lowercase `searchText` field used for search matching.

### Key JavaScript Functions

| Function | Purpose |
|----------|---------|
| `loadData()` | Initializes data from `embeddedData` into `allData` |
| `searchData(searchTerm)` | Full-text search using `String.includes()` |
| `performSearch()` | Main search handler, triggered on input |
| `displayResults(results)` | Renders search results to DOM |
| `renderResult(item)` | Creates HTML for a single result card |
| `highlightText(text, searchTerm)` | Highlights matching terms in results |
| `cleanText(text)` | Formats and sanitizes display text |
| `getTypeBadgeClass(type)` | Maps data source type to CSS badge class |
| `matchesFilter(type)` | Checks if item matches current filter |

### Global State

- `allData` — All loaded items (array)
- `currentFilter` — Active filter type (string)
- `currentSearchTerm` — Current search query (string)

## CSS Conventions

- **Color scheme**: Carta Healthcare branding with CSS custom properties (`--carta-teal-dark`, `--carta-teal`, `--carta-blue`, `--carta-gray-*`, `--carta-accent`)
- **Naming**: BEM-like convention (`.result-item`, `.result-header`, `.type-badge`)
- **Type badges**: `.type-dictionary`, `.type-supporting`, `.type-acd`, `.type-pending`
- **Layout**: Flexbox-based responsive design

## Development Workflow

### Local Development

No build step. Open `index.html` directly in a browser:

```bash
# Extract the zip first
unzip files.zip -d site/
# Then open in browser
open site/index.html    # macOS
xdg-open site/index.html  # Linux
```

### Deployment

Deployment is automatic via GitHub Actions when pushing to `main`:

1. Push changes to `main` branch
2. GitHub Actions workflow (`.github/workflows/deploy.yml`) triggers
3. Entire repo is uploaded as a GitHub Pages artifact
4. Site deploys to `https://<username>.github.io/<repo-name>/`

Manual deployment can be triggered via `workflow_dispatch` in the Actions tab.

### Updating Data

To update searchable data:
1. Modify the `embeddedData` object inside `index.html`
2. Ensure each new item has a `searchText` field (lowercase concatenation of searchable fields)
3. Update `files.zip` with the modified `index.html`
4. Commit and push to `main`

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

3. **Data lives in the HTML** — All 608 items are embedded directly in JavaScript. There is no external API, database, or JSON file.

4. **The zip is the source of truth** — The deployable files live inside `files.zip`. When making changes, the zip contents must be updated.

5. **Branding matters** — Carta Healthcare branding (colors, logo, fonts) must be preserved in any UI changes.

6. **Search is client-side** — All search happens in-browser using `String.includes()`. There is no server-side component.

7. **GitHub Pages deployment** — The deploy workflow expects files in the repo root. The workflow file should be at `.github/workflows/deploy.yml` in the deployed repository.

## Data Source Types

| Type | Badge Class | Count | Fields |
|------|-------------|-------|--------|
| Data Dictionary | `.type-dictionary` | 346 | seqNum, elementName, codingInstructions, targetValue |
| Supporting Definition | `.type-supporting` | 95 | seqNum, elementName, supportingDefinition |
| Additional Coding Directives | `.type-acd` | 145 | seqNum, elementName, question, answer, postedDate |
| Pending Definition Updates | `.type-pending` | 22 | seqNum, elementName, codingInstruction, notes, targetValue |

## Common Tasks

### Adding a new data entry
Add an object to the appropriate array in `embeddedData` with all required fields plus a `searchText` field containing a lowercase concatenation of all searchable text.

### Adding a new data source type
1. Add a new array to `embeddedData`
2. Add a new filter button in the HTML
3. Add CSS for the new type badge (`.type-<name>`)
4. Update `getTypeBadgeClass()` to map the new type
5. Update `matchesFilter()` to handle the new filter
6. Update `renderResult()` to display fields for the new type

### Modifying search behavior
The `searchData()` function performs simple substring matching on the `searchText` field. To change search logic (e.g., fuzzy matching, ranking), modify this function.
