# Starches Documentation

Starches is a suite of tools for publishing [Arches](https://www.archesproject.org/) cultural heritage data as searchable, map-enabled static websites. It combines Hugo (static site generation), Vite (frontend bundling), TypeScript, and Rust (WASM/NAPI).

## Documentation Pages

| Page | Description |
|------|-------------|
| [Getting Started](getting-started.md) | Prerequisites, installation, and running your first instance |
| [Architecture](architecture.md) | System architecture, data flow, and project structure |
| [Configuration](configuration.md) | Hugo config, environment variables, prebuild config, and template config |
| [Templates](templates.md) | Handlebars and Hugo template system for map, asset, and search pages |
| [Data Pipeline](data-pipeline.md) | Data setup, ETL processing, indexing, and filter configuration |
| [Customization](customization.md) | Overriding layouts, theming, map configuration, and adding pages |
| [Development](development.md) | Development workflow, testing, and debugging |
| [Deployment](deployment.md) | Docker builds, CI/CD pipelines, and Azure deployment |

## Quick Reference

```bash
# Install dependencies
npm install

# Fetch data and content from blob storage
npm run fetch:prebuild
npm run fetch:content

# Process data into search indexes
npm run reindex

# Precompile Handlebars templates
npm run precompile:templates

# Start development server (port 1313)
npm start

# Start with Hugo module workspace (parallel repo development)
npm run dev

# Full production build
npm run build:full

# Run tests
npm test
```
