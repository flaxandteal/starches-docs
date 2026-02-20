# Architecture

## Monorepo Structure

Starches is part of a monorepo containing several interconnected projects:

| Project | Purpose | Key Tech |
|---------|---------|----------|
| **starches** | Main static site generator — frontend assets for search, maps, and asset detail pages | TypeScript, Vite, Hugo, Vitest |
| **alizarin** | ORM library for Arches graph-based data models | TypeScript + Rust (WASM via wasm-pack) |
| **starches-builder** | CLI tool for processing heritage data into search indexes and map layers | TypeScript + Rust (NAPI bindings) |
| **quartz-starches** | Reference implementation / consuming Hugo site | Hugo, Cypress (E2E) |

## Data Flow Pipeline

```
Arches JSON exports
  │
  ▼
Alizarin ORM (parse and model graph data)
  │
  ▼
starches-builder ETL
  ├── Pagefind search indexes
  └── FlatGeobuf spatial tiles
  │
  ▼
Hugo + Vite (render static site)
  ├── Search pages (Pagefind full-text search)
  ├── Map pages (MapLibre GL interactive maps)
  └── Asset detail pages (resource information)
```

### Step-by-step

1. **Arches JSON exports** — Raw resource data exported from an Arches instance is placed in `prebuild/business_data/`.
2. **Alizarin ORM** — Parses the Arches graph-based data models. `GraphManager` loads resource model graphs and provides typed access to heritage data. Performance-critical graph operations run in Rust via WASM.
3. **starches-builder ETL** — The CLI tool runs extract-transform-load on the raw data. It generates Pagefind search indexes (for full-text search) and FlatGeobuf spatial tiles (for map rendering). Handlebars markdown templates control what data goes into each search index entry.
4. **Hugo + Vite** — Hugo renders the static HTML pages. Vite bundles the TypeScript frontend assets. The result is a fully static site with search, maps, and asset detail pages.

## Hugo Module System

Starches uses Hugo's module system. The `starches` package is consumed as a Hugo module by downstream sites like `quartz-starches`:

```yaml
# hugo.yaml
module:
  imports:
    - path: 'github.com/flaxandteal/hugo-theme-qld-design-system'
    - path: 'github.com/flaxandteal/starches'
```

Hugo's template lookup order means:
- **Starches** provides base layouts in `starches/layouts/` (partials, shortcodes, default templates)
- **Your site** can override any template by placing a file at the same path in your own `layouts/` directory
- **Theme modules** (e.g. the QLD design system) provide additional layouts and partials

## Directory Layout (quartz-starches)

```
quartz-starches/
├── assets/              # TypeScript entry points (bundled by Vite)
│   ├── asset.ts         # Asset detail page logic
│   ├── map.ts           # Interactive map
│   ├── search.ts        # Pagefind search
│   └── ...
│
├── config/
│   └── _default/
│       └── params.yaml  # Site parameters (auto-generated from blob)
│
├── content/             # Hugo content pages
│   ├── _index.md        # Homepage
│   ├── map.md           # Map/search page
│   └── asset.md         # Asset detail template
│
├── layouts/             # Hugo template overrides (empty = inherit from modules)
│
├── prebuild/            # Data processing
│   ├── business_data/   # Arches JSON exports
│   ├── graphs/          # Resource model schemas
│   ├── reference_data/  # Controlled vocabularies
│   ├── preindex/        # Pre-processed index data (generated)
│   ├── fgb/             # FlatGeobuf spatial tiles (generated)
│   ├── indexTemplates/  # Handlebars templates for indexing
│   └── prebuild.json    # ETL configuration
│
├── static/              # Static assets
│   ├── css/             # Stylesheets
│   ├── js/              # JavaScript (including precompiled templates)
│   ├── templates/       # Handlebars source templates (.md files)
│   └── img/             # Images
│
├── utils/               # Build-time TypeScript scripts
│   ├── preindex.ts      # Pre-process data for indexing
│   ├── reindex.ts       # Main indexing pipeline
│   ├── fetch-content.ts # Fetch content from blob storage
│   ├── fetch-prebuild.ts# Fetch data from blob
│   └── ...
│
├── cypress/             # E2E tests
├── docs/                # Built static site output (Hugo publishDir)
├── hugo.yaml            # Hugo configuration
├── hugo.work            # Hugo workspace for parallel development
└── package.json         # Dependencies and scripts
```

## Frontend Entry Points

Each TypeScript file in `assets/` is a standalone entry point for a page type, bundled by Vite:

| Entry Point | Purpose |
|-------------|---------|
| `asset.ts` | Asset detail pages — renders resource data using alizarin and Handlebars templates |
| `map.ts` | Interactive MapLibre GL maps with FlatGeobuf data layers |
| `search.ts` | Pagefind-based full-text search |
| `searchContext.ts` | Preserves navigation state across search → detail → back |

## Key Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| Hugo Extended | 0.145+ | Static site generation (SCSS support requires Extended edition) |
| Vite | 6.x | Frontend bundling and dev server |
| TypeScript | 5.x | Type-safe frontend and build tooling |
| MapLibre GL | 5.x | Interactive vector maps |
| Pagefind | 1.3.x | Static full-text search |
| FlatGeobuf | 4.x | Geospatial data encoding for map tiles |
| Handlebars | 4.x | Template rendering (result cards, asset pages, map dialogs) |
| Marked | 17.x | Markdown-to-HTML conversion with custom extensions |
| Alizarin | 0.2.x | Arches graph data ORM (TypeScript + Rust/WASM) |
