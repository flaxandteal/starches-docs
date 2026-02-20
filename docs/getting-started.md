# Getting Started

This guide walks you through setting up a new Starches instance from scratch.

## Prerequisites

| Tool | Version | Notes |
|------|---------|-------|
| **Node.js** | 20+ | 22+ recommended. Used for build tooling, ETL, and dev server |
| **Hugo Extended** | 0.145+ | Must be the **Extended** edition (required for SCSS). [Install guide](https://gohugo.io/getting-started/installing/) |
| **Go** | 1.21+ | Required for Hugo modules. [Install guide](https://go.dev/doc/install) |
| **Git** | Latest | Required for Hugo module resolution |
| **Rust** | Latest stable | Only needed if building alizarin WASM locally. [Install guide](https://rustup.rs/) |

## Installation

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd quartz-starches
```

### 2. Install Node Dependencies

```bash
npm install
```

This installs all dependencies including `alizarin`, `starches`, and `starches-builder` from their GitHub release tarballs.

### 3. Initialize Hugo Modules

```bash
hugo mod get
```

This pulls the Hugo theme modules defined in `hugo.yaml` (e.g. the design system theme and starches itself).

## Data Setup

Starches needs Arches data exports to generate the site. There are two ways to get this data:

### Option A: Fetch from Blob Storage (Recommended)

If your project uses Azure Blob Storage for data, create a `.env` file:

```bash
BLOB_BASE_URL=https://your-storage-account.blob.core.windows.net
```

Then fetch the data:

```bash
# Fetch prebuild data (graphs, business data, reference data)
npm run fetch:prebuild

# Fetch site content (Hugo content pages, params config)
npm run fetch:content
```

### Option B: Manual Data Setup

Create the required directory structure and populate it manually:

```
prebuild/
  business_data/       # Arches JSON resource exports
  graphs/
    resource_models/   # Resource model definition JSON files
    branches/          # Branch definition JSON files
  reference_data/
    collections/       # Controlled vocabulary collection JSON files
  preindex/            # (created automatically by the pipeline)
  fgb/                 # (created automatically by the pipeline)
  indexTemplates/      # Handlebars templates for search indexing
  prebuild.json        # ETL configuration (see Configuration docs)
  graphs.json          # Graph definitions
  permissions.json     # Access control rules
```

You also need Hugo content pages in `content/`:

- `_index.md` — Homepage
- `map.md` — Map/search page (see [Templates](templates.md) for front matter reference)
- `asset.md` — Asset detail page template

And site parameters in `config/_default/params.yaml`.

## Build the Data Pipeline

Once your data is in place, process it into search indexes and map tiles:

```bash
# Precompile Handlebars templates
npm run precompile:templates

# Process business data into Pagefind indexes and FlatGeobuf tiles
npm run reindex
```

## Start the Development Server

```bash
# Standard Hugo server on localhost:1313
npm start

# Skip blob fetch (use local data only)
npm run start:local
```

Open [http://localhost:1313](http://localhost:1313) to view your site.

## Setup Checklist

- [ ] Node.js 20+, Hugo Extended, Go, and Git installed
- [ ] Repository cloned and `npm install` completed
- [ ] Hugo modules initialized (`hugo mod get`)
- [ ] `.env` file created with `BLOB_BASE_URL` (if using blob storage)
- [ ] Prebuild data available (fetched or manually placed)
- [ ] Content pages available in `content/`
- [ ] Site params configured in `config/_default/params.yaml`
- [ ] `prebuild.json` configured with data sources
- [ ] Templates precompiled (`npm run precompile:templates`)
- [ ] Data processed (`npm run reindex`)
- [ ] Dev server running (`npm start`)
