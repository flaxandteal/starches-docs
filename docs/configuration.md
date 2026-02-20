# Configuration Reference

This page covers all configuration files used by a Starches instance.

## Hugo Configuration (`hugo.yaml`)

The main Hugo configuration file. Key settings:

```yaml
baseURL: '/'
languageCode: 'en-gb'
title: 'Heritage Register'
publishDir: "docs"

params:
  version: "0.3.8"
  environment: "uat"          # uat | production
  pagefind:
    useCustomInput: true      # Use custom search input (not Pagefind default)

markup:
  goldmark:
    renderer:
      unsafe: true            # Allow raw HTML in markdown

module:
  imports:
    - path: 'github.com/flaxandteal/hugo-theme-qld-design-system'
    - path: 'github.com/flaxandteal/starches'
  mounts:
    - source: static
      target: static
    - source: node_modules/alizarin/dist/alizarin_bg.wasm
      target: static/wasm/alizarin_bg.wasm

security:
  funcs:
    getenv:
      - '^BLOB_'
      - '^HUGO_'
```

### Key Settings

- **`publishDir`** — Output directory for the built site. Set to `docs` by default.
- **`module.imports`** — Hugo modules to import. This is where you add your theme and the starches module.
- **`module.mounts`** — Mount points map source files into Hugo's virtual filesystem. The alizarin WASM binary must be mounted to `static/wasm/`.
- **`security.funcs.getenv`** — Allowlisted environment variable prefixes that Hugo templates can read.

## Site Parameters (`config/_default/params.yaml`)

Site-specific parameters loaded by Hugo. This file is typically auto-generated from blob storage by `fetch-content.ts` and is **gitignored**. It contains site-specific text, branding, and configuration that varies per deployment.

## Environment Variables

Create a `.env` file in the project root:

| Variable | Required | Description |
|----------|----------|-------------|
| `BLOB_BASE_URL` | Yes (for blob fetch) | Azure Blob Storage base URL for fetching content and prebuild data |
| `SKIP_BLOB_FETCH` | No | Set to `true` to skip blob storage downloads (use local data only) |
| `DATA_FILE` | No | Specifies which business data JSON file to process (default: `test_data.json` in Docker) |
| `PAGEFIND_BINARY_PATH` | No | Path to the Pagefind binary (default: `./pagefind-bin`) |
| `VITE_DEBUG` | No | Set to `true` for debug logging in the frontend (auto-set in `.env.development`) |
| `STARCHES_INCLUDE_PRIVATE` | No | Set to `1` to include private/non-public resources |
| `HUGO_MODULE_WORKSPACE` | No | Path to Hugo workspace file for parallel development (set by `npm run dev`) |

## Prebuild Configuration (`prebuild.json`)

Controls the ETL data pipeline. Located in `prebuild/prebuild.json`.

```json
{
    "paths": {
        "geometry": "location_data.geometry.0.geospatial_coordinates",
        "location": "location_data.geometry.0.geospatial_coordinates"
    },
    "filters": [
        {
            "name": "Category",
            "graph": "HeritageItem",
            "type": "array",
            "dynamic": true,
            "path": "construction_phases.*.phase_classification.monument_type.*.ancestors.0"
        }
    ],
    "indexTemplates": {
        "HeritageItem": "heritage-asset-index-hb.md",
        "_unknown": "_unknown-index-hb.md"
    },
    "customDatatypes": {
        "tm65centrepoint": "non-localized-string"
    },
    "sources": [
        {
            "resources": "prebuild/business_data/registries.json",
            "public": true,
            "searchFor": ["3a6ce8b9-0357-4a72-b9a9-d8fdced04360"],
            "slugPrefix": "REG_"
        }
    ],
    "thumbnail": [
        {
            "graph": "HeritageItem",
            "path": "images"
        }
    ],
    "indexCharacters": 2000,
    "indexCharactersWarnOnly": true
}
```

### Field Reference

| Field | Description |
|-------|-------------|
| `paths.geometry` | Alizarin path to geometry data for map rendering |
| `paths.location` | Alizarin path to location coordinates |
| `filters` | Filter definitions for search faceting (see [Data Pipeline](data-pipeline.md)) |
| `indexTemplates` | Map of graph name to Handlebars template for search index generation |
| `customDatatypes` | Override Arches datatype handling for specific types |
| `sources` | Data source definitions (see below) |
| `thumbnail` | Thumbnail image path configuration per graph type |
| `indexCharacters` | Maximum characters per search index entry |
| `indexCharactersWarnOnly` | If `true`, log a warning instead of erroring when index character limit is exceeded |

### Sources

Each entry in the `sources` array defines a data file to process:

| Field | Description |
|-------|-------------|
| `resources` | Path to the Arches JSON export file (supports regex patterns like `buildings_.*.json`) |
| `public` | Whether the resources are publicly accessible |
| `slugPrefix` | Prefix added to generated URL slugs (e.g. `REG_`, `AAI_`) |
| `searchFor` | Array of specific resource model UUIDs to process from this file |
| `dependencies` | Array of additional data files needed for related resources |

## Template Precompilation (`starches.templates.json`)

Controls which Handlebars templates are precompiled for the frontend. Located in the project root.

```json
{
  "templateSources": [
    {
      "module": "github.com/flaxandteal/hugo-theme-qld-design-system",
      "dir": "static/templates",
      "htmlTemplates": [
        "result-card-template.html",
        "filter-list-template.html",
        "asset-nodegroup-template.html",
        "map-dialog-template.html"
      ]
    },
    {
      "dir": "static/templates",
      "mdTemplates": [
        "heritage-asset-public-hb.md",
        "activity.md"
      ]
    }
  ],
  "cdnBase": "https://cdn.jsdelivr.net/npm/@qld-gov-au/qgds-bootstrap5@2.0.9/dist/...",
  "components": [
    {
      "name": "header",
      "partials": ["headerBrand"]
    }
  ],
  "outputDir": "static/js",
  "outputFile": "precompiled-templates.js"
}
```

### Field Reference

| Field | Description |
|-------|-------------|
| `templateSources[].module` | Hugo module path that contains the templates (for module-installed themes) |
| `templateSources[].dir` | Directory within the module (or project) containing the templates |
| `templateSources[].localpath` | Alternative to `module` — local filesystem path for the theme |
| `templateSources[].htmlTemplates` | List of HTML Handlebars templates to precompile |
| `templateSources[].mdTemplates` | List of Markdown Handlebars templates to precompile |
| `cdnBase` | CDN URL for components that need to be fetched and compiled |
| `components` | CDN components and their partials to include |
| `outputDir` | Output directory for compiled templates (keep as `static/js`) |
| `outputFile` | Output filename (keep as `precompiled-templates.js`) |

## Content Front Matter

Hugo content pages use YAML front matter to configure each page. See [Templates](templates.md) for the full front matter reference for map and asset pages.
