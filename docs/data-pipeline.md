# Data Pipeline

This page covers how heritage data flows from Arches JSON exports into the search indexes and map tiles that power the static site.

## Required Data

### Directory Structure

```
prebuild/
├── business_data/           # Arches JSON resource exports
│   ├── registries.json
│   ├── aai_merged.json
│   └── buildings_*.json     # Supports regex patterns
│
├── graphs/
│   ├── resource_models/     # Resource model definition files
│   └── branches/            # Branch definition files
│
├── reference_data/
│   └── collections/         # Controlled vocabulary collections
│
├── indexTemplates/           # Handlebars templates for search indexing
│
├── preindex/                # Generated: pre-processed index data
├── fgb/                     # Generated: FlatGeobuf spatial tiles
│
├── prebuild.json            # ETL configuration
├── graphs.json              # Graph definitions list
└── permissions.json         # Access control rules
```

### Data Sources

**Business data** (`business_data/`) — JSON files exported from Arches containing the actual heritage resource data. Each file can contain resources from one or more resource models.

**Graphs** (`graphs/`) — Resource model and branch definitions that describe the structure of the heritage data. These define the node hierarchy that Alizarin traverses.

**Reference data** (`reference_data/collections/`) — Controlled vocabularies (concept collections) used for lookups like place types, heritage categories, etc.

## ETL Pipeline

The ETL (Extract-Transform-Load) process is run by `starches-builder`:

### Step 1: ETL — Extract and Transform

```bash
npx starches-builder etl --file ./prebuild/business_data/data.json --prefix qld- --summary
```

This step:
1. Loads the resource model graphs via Alizarin
2. Reads the business data JSON files specified in `prebuild.json` sources
3. Processes each resource through the Handlebars index template (defined in `prebuild.json` → `indexTemplates`)
4. Generates pre-processed data in `prebuild/preindex/`
5. Generates FlatGeobuf spatial tiles in `prebuild/fgb/` using the geometry paths from `prebuild.json`

### Step 2: Index — Build Search Indexes

```bash
npx starches-builder index --site docs
```

This step:
1. Reads the pre-processed data from `prebuild/preindex/`
2. Generates Pagefind search indexes in the `docs/` output directory
3. The indexes enable full-text search on the static site

### Combined Pipeline

The `reindex` npm script runs the full pipeline:

```bash
npm run reindex
```

This clears existing indexes, pre-processes each business data file, then runs the full reindex.

## Source Configuration

Each entry in `prebuild.json` → `sources` defines a data file to process:

```json
{
    "resources": "prebuild/business_data/registries.json",
    "public": true,
    "searchFor": ["3a6ce8b9-0357-4a72-b9a9-d8fdced04360"],
    "slugPrefix": "REG_",
    "dependencies": [
        "prebuild/business_data/persons_merged.json"
    ]
}
```

| Field | Description |
|-------|-------------|
| `resources` | Path to the JSON file. Supports regex patterns (e.g. `buildings_.*.json`) |
| `public` | Whether these resources are publicly accessible |
| `slugPrefix` | Prefix for generated URL slugs (e.g. `REG_` produces slugs like `REG_12345`) |
| `searchFor` | Optional: only process resources matching these resource model UUIDs |
| `dependencies` | Additional JSON files needed for related/linked resources |

## Geometry Configuration

The `paths` section in `prebuild.json` tells the pipeline where to find spatial data:

```json
{
    "paths": {
        "geometry": "location_data.geometry.0.geospatial_coordinates",
        "location": "location_data.geometry.0.geospatial_coordinates"
    }
}
```

These are Alizarin paths that chain through the resource graph to reach the geometry node. The extracted coordinates are used to generate FlatGeobuf map tiles.

## Index Templates

Index templates are Handlebars markdown files that define what data goes into each search index entry. They are mapped per graph type in `prebuild.json`:

```json
{
    "indexTemplates": {
        "HeritageItem": "heritage-asset-index-hb.md",
        "_unknown": "_unknown-index-hb.md"
    }
}
```

- The key is the graph/resource model name
- The value is the template filename (located in `prebuild/indexTemplates/`)
- `_unknown` is a fallback for resources that don't match any defined graph

These templates use the same Alizarin syntax as asset page markdown templates (see [Templates](templates.md#markdown-templates)).

## Thumbnail Configuration

```json
{
    "thumbnail": [
        {
            "graph": "HeritageItem",
            "path": "images"
        }
    ]
}
```

Defines how to extract thumbnail images for each graph type, used in search result cards and map dialogs.

## Filter Configuration

Filters allow users to facet search results by categories. Each filter is defined in `prebuild.json`:

```json
{
    "filters": [
        {
            "name": "Category",
            "graph": "HeritageItem",
            "type": "array",
            "dynamic": true,
            "path": "construction_phases.*.phase_classification.monument_type.*.ancestors.0"
        }
    ]
}
```

| Field | Description |
|-------|-------------|
| `name` | Display name of the filter category |
| `graph` | Which resource model's resources this filter applies to |
| `type` | `"single"` for single-value nodes, `"array"` for multi-value nodes |
| `dynamic` | `true` to use dynamic Alizarin output (needed for concept UUIDs), `false` for static output (faster) |
| `path` | Alizarin path to the filter value. Use `*` for cardinality-n traversal |

**Path details:**
- Use `*` to traverse through cardinality-n nodes (e.g. `construction_phases.*`)
- Use `ancestors.0` to group by the parent concept in the hierarchy
- The path determines what values appear as filter options

The frontend rendering of each filter option is controlled by the `filter-list-template.html` Handlebars template (see [Templates](templates.md)).
