# Customization

This page covers how to customize the look, feel, and behavior of your Starches site.

## Hugo Template Overrides

Starches provides default layouts via the Hugo module system. To override any layout, create a file at the **same path** in your project's `layouts/` directory. Hugo's template lookup order gives your local files priority over module-provided ones.

For example, to override the map partial:

```
# Starches provides:
starches/layouts/partials/starches/map.html

# Override by creating:
your-site/layouts/partials/starches/map.html
```

### Available Layouts to Override

| Layout | Path | Purpose |
|--------|------|---------|
| Map partial | `layouts/partials/starches/map.html` | Map + search + results layout |
| Asset partial | `layouts/partials/starches/asset.html` | Asset detail page content |
| Map search page | `layouts/_default/map-search.html` | Full map page wrapper (areas above/below map and search) |
| Asset page | `layouts/_default/asset.html` | Full asset detail page wrapper |
| Head partial | `layouts/partials/starches/head.html` | `<head>` section additions |
| Map dialog | `layouts/partials/starches/map-dialog.html` | Map popup structure |
| Map filters | `layouts/partials/starches/map-filters.html` | Filter controls layout |

## Handlebars Template Customization

To customize dynamic content (search results, asset node groups, map popups), create or modify the Handlebars templates in `static/templates/`:

| Template | Purpose |
|----------|---------|
| `result-card-template.html` | Search result card layout and styling |
| `asset-nodegroup-template.html` | Node group display on asset detail pages |
| `map-dialog-template.html` | Map marker popup content |
| `filter-list-template.html` | Individual filter option (e.g. radio button + label) |
| `heritage-asset-public-hb.md` | Asset page content structure (sections and node blocks) |

If your theme module provides these templates, you can override them by placing versions in your own `static/templates/` directory and updating `starches.templates.json` accordingly.

See [Templates](templates.md) for the full variable reference for each template.

## Adding Content Pages

Create Hugo content pages in the `content/` directory. Each page uses YAML front matter to configure its behavior.

### Map Page

```yaml
---
title: 'Heritage Register'
tabTitle: 'Search: Heritage Register'
url: '/map'
type: '_default'
layout: 'map-search'
includeSearch: true
center: "[144, -22.5]"
zoom: 4.5
mobileZoom: 3
search_config:
  useCustomInput: true
---

Optional page content appears below the map.
```

### Additional Pages

Standard Hugo content pages work as normal. Create markdown files in `content/` with appropriate front matter:

```yaml
---
title: 'About'
url: '/about'
---

Page content here.
```

## Map Configuration

Map basemaps and overlays are configured in the map page's front matter under `map_config`:

### Basemaps

```yaml
map_config:
  defaultBasemap: "vector"
  basemaps:
    - id: "vector"
      label: "Map"
      config:
        type: "arcgis-vector"
        url: "https://your-vector-tile-server/VectorTileServer"
        attribution: "© Your Attribution"
    - id: "aerial"
      label: "Aerial"
      config:
        type: "raster"
        tiles: "https://your-tile-server/tile/{z}/{y}/{x}"
        attribution: "© Your Attribution"
```

**Supported basemap types:**
- `arcgis-vector` — ArcGIS vector tile service (provide `url`)
- `raster` — Standard XYZ raster tiles (provide `tiles` with `{z}/{y}/{x}` placeholders)

### Overlays

```yaml
  overlays:
    - id: "cadastre"
      label: "Cadastral Parcels"
      visible: false
      config:
        type: "wms-export"
        tiles: "https://your-server/MapServer/export?bbox={bbox-epsg-3857}&bboxSR=3857&imageSR=3857&size=256,256&format=png32&transparent=true&layers=show:4&f=image"
```

**Supported overlay types:**
- `wms-export` — ArcGIS MapServer export endpoint with `{bbox-epsg-3857}` placeholder

Set `visible: false` to have overlays hidden by default (toggled by the user via map controls).

### Map Center and Zoom

```yaml
center: "[longitude, latitude]"    # Map center as JSON array string
zoom: 4.5                          # Default zoom level
mobileZoom: 3                      # Zoom level for mobile devices
```

## CSS Customization

Add custom stylesheets in `static/css/`. Reference them from your Hugo layouts or head partial.

The reference implementation uses:
- `static/css/custom.css` — Main custom styles
- `static/css/asset.css` — Asset page styles
- `static/css/map.css` — Map page styles
- `static/css/map-controls.css` — Map control styles

## Breadcrumb Navigation

Add breadcrumbs to any page via front matter:

```yaml
breadcrumbs:
  - name: 'Home'
    url: '/'
  - name: 'Search'
    active: 'true'
```

Set `active: 'true'` on the current page's breadcrumb entry.

## Custom Handlebars Helpers

Additional Handlebars helpers can be defined in `static/js/handlebars-helpers.js`. These are available in all Handlebars templates.

## Asset Page Content Structure

The content displayed on asset detail pages is controlled by the markdown template (e.g. `heritage-asset-public-hb.md`). To customize which data appears and how it's organized:

1. Edit the markdown template in `static/templates/`
2. Use sections (`<!--section:name-->`) to organize content into page areas
3. Use node blocks (`::Title::` ... `::end::`) to define field groups
4. Use Alizarin paths (`{{ ha.node.chain }}`) to extract data

See [Templates — Markdown Templates](templates.md#markdown-templates) for the full syntax reference.
