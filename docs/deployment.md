# Deployment

This page covers building for production, Docker containerization, and CI/CD deployment.

## Production Build

### Full Build

```bash
npm run build:full
```

This runs the complete build pipeline:
1. Fetches all data from blob storage
2. Precompiles Handlebars templates
3. Builds the static site with Hugo (`hugo --minify`)

The output is written to `docs/`.

### Manual Build Steps

If you prefer to run each step individually:

```bash
# 1. Fetch data (if using blob storage)
npm run fetch:prebuild
npm run fetch:content

# 2. Process data
npm run reindex

# 3. Precompile templates
npm run precompile:templates

# 4. Build the static site
hugo --minify
```

## Docker

### Dockerfile

The project includes a multi-stage Dockerfile:

**Stage 1: Build** (Node 23.10.0)
1. Installs npm dependencies
2. Installs Go 1.23.4 and Hugo Extended 0.152.2
3. Runs `starches-builder etl` to process business data
4. Runs `starches-builder index` to generate Pagefind indexes
5. Precompiles Handlebars templates
6. Builds the site with `hugo mod get && hugo`

**Stage 2: Serve** (Nginx Alpine unprivileged)
1. Copies the built `docs/` directory to Nginx's web root
2. Serves on port 8080 as an unprivileged user

### Build Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `DATA_FILE` | `test_data.json` | Business data JSON filename to process |
| `BLOB_BASE_URL` | (none) | Azure Blob Storage URL |

### Docker Compose

```bash
# Build and run locally
docker compose up --build
```

The site is available at [http://localhost:8080](http://localhost:8080).

Ensure `BLOB_BASE_URL` is set in your `.env` file or passed as an environment variable.

### docker-compose.yml

```yaml
version: '3.8'
services:
  web:
    build:
      context: .
      args:
        BLOB_BASE_URL: ${BLOB_BASE_URL}
    ports:
      - "8080:8080"
    user: "33"
```

## CI/CD

### Build and Test Workflow

**File:** `.github/workflows/build-test.yml`
**Trigger:** All branch pushes

1. **Build Docker Image**
   - Downloads prebuild tarball from blob storage
   - Sets up Node.js 22 and installs dependencies
   - Fetches site content from blob
   - Builds and pushes Docker image to `ghcr.io`

2. **Test**
   - Pulls the built Docker image
   - Extracts the `docs/` directory from the container
   - Runs CSS linting (`stylelint`)
   - Runs Cypress E2E tests against the static build (served on port 1314)
   - Uploads screenshots as artifacts on test failure

### Deploy Workflow

**File:** `.github/workflows/deploy.yml`
**Trigger:** Successful build-test on the `dev` branch

1. Pulls the Docker image from `ghcr.io`
2. Extracts the `docs/` directory
3. Logs in to Azure using OIDC
4. Uploads the static site to Azure Storage `$web` container
5. Logs out from Azure

### Required GitHub Secrets

| Secret | Description |
|--------|-------------|
| `BLOB_BASE_URL` | Azure Blob Storage base URL |
| `PREBUILD_TARBALL` | Filename of the prebuild tarball in blob storage |
| `DOCKER_REGISTRY_TOKEN` | GitHub Container Registry (ghcr.io) access token |
| `DOCKER_USERNAME` | Docker registry username |
| `AZURE_CLIENT_ID` | Azure OIDC client ID |
| `AZURE_TENANT_ID` | Azure AD tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Azure subscription ID |
| `AZURE_STORAGE_ACCOUNT` | Azure Storage account name |

## Azure Static Web Apps

The site is deployed to Azure Storage as a static website. Routing is configured in `staticwebapp.config.json`:

- Trailing slash handling: auto
- Navigation fallback: `/index.html`
- Excludes static assets, pagefind files, and images from the fallback
- JSON files served with `application/json` MIME type

## Hosting Alternatives

Since the output is a static site in `docs/`, it can be hosted on any static file server:

- **Nginx** (included in Docker image)
- **Azure Storage Static Website** (CI/CD default)
- **GitHub Pages** (point to `docs/` directory)
- **Netlify / Vercel / Cloudflare Pages** (deploy the `docs/` output)
- **Any web server** capable of serving static files
