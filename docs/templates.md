# Template System

Starches uses a combination of Hugo layouts and Handlebars templates to render pages. Hugo handles the overall page structure, while Handlebars templates control the dynamic content (search results, asset data, map dialogs).

## Overview

There are two categories of templates:

1. **Hugo layouts** — HTML page structures in `layouts/`. Starches provides defaults; your site can override them.
2. **Handlebars templates** — Dynamic content templates in `static/templates/`. These are precompiled into JavaScript and rendered client-side.

### Provided Hugo Layouts

| Layout | Path | Purpose |
|--------|------|---------|
| Map partial | `partials/starches/map.html` | Map and search results layout |
| Asset partial | `partials/starches/asset.html` | Asset detail page layout |
| Map search page | `_default/map-search.html` | Full map page with content areas above/below the map and search |
| Asset page | `_default/asset.html` | Full asset detail page |

These can be used as-is or overridden by placing a file at the same path in your project's `layouts/` directory.

### Handlebars Templates

| Template | Purpose |
|----------|---------|
| `result-card-template.html` | Individual search result card layout |
| `asset-nodegroup-template.html` | Node group display on asset pages |
| `map-dialog-template.html` | Map popup when clicking a marker |
| `filter-list-template.html` | Individual filter pill entry |
| `heritage-asset-public-hb.md` | Markdown template defining asset page content and sections |

---

## Map Page (`map.html`)

The map page combines an interactive map with search functionality and a results list.

### Required Elements

When overriding the map layout, these elements **must** be present:

#### Search Input

Pagefind attaches to an element with `id="search"`. You can use either:

- An `<input>` tag — for a custom-styled search input:
  ```html
  <input id="search" type="text" placeholder="Search..." />
  ```
- A `<div>` tag — Pagefind will inject its default search UI:
  ```html
  <div id="search"></div>
  ```

Without this element, search will not function.

#### Results Container

```html
<div id="results"></div>
```

Search results are injected into this element. You must also provide a `result-card-template.html` (see below) to control how individual results are displayed.

### Content Page Front Matter

The map page is configured via Hugo front matter in `content/map.md`:

```yaml
---
title: 'Heritage Register'
tabTitle: 'Search: Heritage Register'
url: '/map'                      # Route for the map page (default: /map)
type: '_default'
layout: 'map-search'             # Which Hugo layout to use
includeSearch: true
center: "[144, -22.5]"           # Map center [longitude, latitude]
zoom: 4.5                        # Default zoom level
mobileZoom: 3                    # Zoom level on mobile devices
breadcrumbs:                     # Optional breadcrumb navigation
  - name: 'Home'
    url: '/'
  - name: 'Search'
    active: 'true'
search_config:
  useCustomInput: true           # Use custom input (not Pagefind default)
map_config:
  defaultBasemap: "vector"       # ID of the default basemap
  basemaps:                      # Available basemap layers
    - id: "vector"
      label: "Map"
      config:
        type: "arcgis-vector"    # Basemap type
        url: "https://..."       # Tile server URL
        attribution: "..."
    - id: "aerial"
      label: "Aerial"
      config:
        type: "raster"
        tiles: "https://.../{z}/{y}/{x}"
        attribution: "..."
  overlays:                      # Optional overlay layers
    - id: "cadastre"
      label: "Cadastral Parcels"
      visible: false             # Hidden by default
      config:
        type: "wms-export"
        tiles: "https://...?bbox={bbox-epsg-3857}&..."
---

Content below front matter appears on the page (e.g. cultural advice notices).
```

### Map Configuration Options

**Basemap types:**
- `arcgis-vector` — ArcGIS vector tile server
- `raster` — Standard raster tile URL with `{z}/{y}/{x}` placeholders

**Overlay types:**
- `wms-export` — WMS/ArcGIS export endpoint with `{bbox-epsg-3857}` placeholder

---

## Asset Page (`asset.html`)

The asset page displays detailed information about a single heritage resource. Content is organized into **sections**, each containing **node groups** with field data.

### Sections

Each section attaches to a `<div>` on the page. The section name maps to an element ID with the `asset-` prefix:

```html
<div id="asset-overview"></div>
<div id="asset-details"></div>
<div id="asset-history"></div>
```

The content for each section is defined in the markdown template (see [Markdown Templates](#markdown-templates) below).

### Node Group Template (`asset-nodegroup-template.html`)

This Handlebars template controls how each group of related fields is displayed within a section. Available variables:

| Variable | Description |
|----------|-------------|
| `title` | Title of the group (set in the markdown template) |
| `id` | Identifier for the group |
| `icon` | Icon class string (if set in the markdown title) |
| `slug` | Identifier for additional resources |
| `fields` | Array of field data (see below) |
| `body` | If `fields` is not set, contains the raw value text |

**Fields array entries:**

| Variable | Description |
|----------|-------------|
| `labels` | The node name (from graph) or a custom label |
| `value` | The output value of the node |

---

## Result Card Template (`result-card-template.html`)

Controls the layout and styling of individual search result cards displayed in the results list.

This is a Handlebars template stored in `static/templates/`. Available variables:

| Variable | Description |
|----------|-------------|
| `url` | URL to the asset detail page for this resource |
| `slug` | Slug identifier for the resource |
| `title` | Resource title or display name |
| `thumbnailURL` | URL for the thumbnail image (if available) |
| `excerpt` | Text generated during indexing from the index template |
| `location` | Coordinates of the resource on the map |

---

## Map Dialog Template (`map-dialog-template.html`)

Controls the popup that appears when clicking a marker on the map.

This is a Handlebars template stored in `static/templates/`. Available variables:

| Variable | Description |
|----------|-------------|
| `url` | URL to the asset detail page |
| `title` | Resource title or display name |
| `excerpt` | Text generated during indexing |
| `location` | Coordinates of the resource |

---

## Filter List Template (`filter-list-template.html`)

Controls the layout of individual filter entries (e.g. a radio button and label for each filter option). Stored in `static/templates/`.

---

## Markdown Templates

Markdown templates (e.g. `heritage-asset-public-hb.md`) define the content structure for asset pages. They are parsed by Alizarin and Marked to extract data from heritage resources and convert it to HTML.

These templates are stored in `static/templates/` and listed in `starches.templates.json` under `mdTemplates`.

### Sections

Sections split the template content into named groups that attach to elements on the page:

```html
<!--section:asset-overview-->
```

Everything below this marker is added to the `asset-overview` section (rendered into the `<div id="asset-overview">` element).

Define multiple sections to organize content across the page:

```html
<!--section:asset-overview-->
... overview content ...

<!--section:asset-details-->
... details content ...
```

### Node Blocks

Node blocks define a group of fields to display together. They use Alizarin to extract values from the resource graph.

#### Basic syntax

```
::Title::
[@node] {{ ha.monument_names.monument_name }}
[Custom Label] {{ ha.node_semantic.node }}
::end::
```

- **`::Title::`** — Defines the group title. Optionally add an icon: `::Title{icon-class}::`
- **`[@node]`** — Label pulled from the graph node name (the `@` prefix means "use the node name")
- **`[Custom Label]`** — A manually defined label
- **`{{ ha.node_semantic.node }}`** — Alizarin expression to extract the node value. Starts at `ha` (the resource) and chains through the node hierarchy.
- **`::end::`** — Closes the node block

#### Handling cardinality-n nodes

When a node has multiple values (cardinality n), you cannot chain directly through it. Use `#each` to loop:

```
::Item Type::
{{#each ha.construction_phases}}
{{#if phase_classification.monument_type }}
{{#each phase_classification.monument_type}}
[Place Type] {{{ . }}}
{{/each}}
{{/if}}
{{/each}}
::end::
```

- **`{{#each ha.construction_phases}}`** — Loop through all construction phases
- **`{{#if ...}}`** — Conditional check for the existence of a value
- **`{{{ . }}}`** — Output the current loop value (triple braces for unescaped HTML)

---

## Template Precompilation

All Handlebars templates must be precompiled before Hugo builds the site. This is controlled by `starches.templates.json` (see [Configuration](configuration.md)) and run with:

```bash
npm run precompile:templates
```

This compiles all listed HTML and Markdown templates into a single `static/js/precompiled-templates.js` file that is loaded by the frontend.
