# Development

This page covers the development workflow, testing, and debugging for a Starches instance.

## Development Server

### Standard Mode

```bash
npm start
```

Runs Hugo server on [http://localhost:1313](http://localhost:1313) with draft pages enabled.

### Local-Only Mode

```bash
npm run start:local
```

Same as above but skips any blob storage fetch. Use this when you already have all data locally.

### Production Preview

```bash
npm run start:prod
```

Runs the Hugo server in production mode (no drafts, production environment settings).

### Hugo Module Workspace Mode

```bash
npm run dev
```

Runs Hugo with `HUGO_MODULE_WORKSPACE=hugo.work`, which enables parallel development across the monorepo. The `hugo.work` file maps local directories instead of fetching modules from Git:

```
go 1.24.6

use .
use ../hugo-theme-qld-design-system
use ../starches
```

This means changes to the `starches` or theme module are picked up immediately without needing to push and update module versions.

## Working with Parallel Repos

For active development on starches and its dependencies:

1. Clone the related repos as siblings:
   ```
   parent-dir/
   ├── quartz-starches/
   ├── starches/
   └── hugo-theme-qld-design-system/
   ```

2. Use `npm run dev` in quartz-starches to activate the workspace

3. Changes in `starches/layouts/` or `starches/assets/` are reflected immediately on reload

## Rebuilding After Changes

### Template Changes

If you modify Handlebars templates in `static/templates/`, recompile:

```bash
npm run precompile:templates
```

Then restart the Hugo server.

### Data Changes

If you modify business data or `prebuild.json`, re-run the data pipeline:

```bash
npm run reindex
```

## Testing

### CSS Linting

```bash
npm run test:lint:css
```

Runs Stylelint against all CSS files in `static/` and `themes/`.

### Cypress E2E Tests

```bash
# Start server and run tests automatically
npm run test:cy:dev

# Or run interactively
npm run cy:open
```

Cypress tests are in `cypress/e2e/`:
- `asset.spec.js` — Asset detail page tests (tabs, carousel, map, PDF export)
- `layout.spec.js` — Layout tests, navigation, WCAG accessibility checks (via axe-core)

The Cypress config (`cypress.config.js`) uses `http://localhost:1313` as the base URL for local testing. CI uses port 1314 with a static file server.

### Unit Tests

```bash
npm run vite-test
```

Runs Vitest unit tests. For interactive mode:

```bash
npm run test:ui
```

### Full Test Suite

```bash
npm test
```

Runs CSS linting followed by the full Cypress E2E suite.

## Debugging

### Frontend Debug Logging

The starches frontend uses `debug()`, `debugWarn()`, and `debugError()` from `utils/debug.ts` instead of `console.log`. These are:
- **Enabled** in development (when `VITE_DEBUG=true`)
- **Stripped** in production builds

Use these functions in any frontend TypeScript code to get development-only console output.

### Environment-Based Debug Settings

Development mode is configured via `.env.development` in the starches package:

```
VITE_DEBUG=true
STARCHES_INCLUDE_PRIVATE=0
```

### Path Aliases

When working with TypeScript imports:
- `@` resolves to `./assets/` (in starches and quartz-starches)
- `@utils` resolves to `./utils/` (in starches)

## Code Style

- TypeScript strict mode across all projects
- ES Modules (`"type": "module"` in all package.json files)
- 2-space indentation, semicolons
- PascalCase for classes, camelCase for variables and functions
- Use `debug()` / `debugWarn()` / `debugError()` instead of `console.log`
