# Architecture

## Monorepo Structure

The project follows a standard Backstage monorepo layout managed by Yarn 4 workspaces.

```
ca-backstage-demo/
  packages/
    app/          # Frontend React SPA
    backend/      # Backend Node.js service
  plugins/        # Custom Backstage plugins (empty so far)
  examples/       # Catalog data and templates
    entities.yaml
    org.yaml
    template/     # Node.js example template (scaffolder)
```

## Frontend (packages/app)

- Entry: packages/app/src/index.tsx -> App.tsx
- React SPA using @backstage/app-defaults createApp()
- Sign-in via GitHub OAuth (githubAuthApiRef), configured in App.tsx
- Route structure defined in App.tsx using FlatRoutes:
  - / -> redirects to /catalog
  - /catalog -> CatalogIndexPage
  - /catalog/:namespace/:kind/:name -> CatalogEntityPage with EntityPage.tsx
  - /docs -> TechDocsIndexPage
  - /create -> ScaffolderPage (template creation wizard)
  - /api-docs -> ApiExplorerPage
  - /catalog-import -> CatalogImportPage (permission-gated)
  - /search -> SearchPage
  - /settings -> UserSettingsPage
  - /catalog-graph -> CatalogGraphPage
  - /notifications -> NotificationsPage
- Sidebar navigation in Root.tsx: Home, My Groups, APIs, Docs, Create..., Notifications, Settings
- EntityPage.tsx handles entity detail views by kind (Component, API, User, Group, System, Domain)
  - Service components get CI/CD, Kubernetes, API, Dependencies, Docs tabs
  - CI/CD tab is currently a placeholder (EmptyState)

## Backend (packages/backend)

- Entry: packages/backend/src/index.ts
- Uses @backstage/backend-defaults createBackend() (new backend system)
- Registered plugins:
  - app-backend (serves frontend)
  - proxy-backend
  - scaffolder-backend + github module + notifications module
  - techdocs-backend
  - auth-backend + github-provider module
  - catalog-backend + scaffolder-entity-model + logs modules
  - permission-backend + allow-all-policy module
  - search-backend + pg engine + catalog collator + techdocs collator
  - kubernetes-backend
  - notifications-backend + signals-backend

## Configuration

- app-config.yaml: Development config
  - SQLite in-memory database
  - GitHub integration via GITHUB_TOKEN env var
  - GitHub OAuth auth (clientId/clientSecret hardcoded for dev)
  - Guest auth enabled
  - TechDocs: local builder, docker generator, local publisher
  - Catalog sources: examples/entities.yaml, examples/template/template.yaml, examples/org.yaml
  - Kubernetes plugin enabled (no clusters configured yet)
  - Permissions enabled

- app-config.production.yaml: Production overrides
  - PostgreSQL database (env vars: POSTGRES_HOST, POSTGRES_PORT, POSTGRES_USER, POSTGRES_PASSWORD)
  - Catalog sources same as dev (still pointing to examples)

## Catalog Data (examples/)

- entities.yaml: One System ("examples"), one Component ("example-website"), one API ("example-grpc-api")
- org.yaml: Three Users (guest, francojeferson, leoanjossantos), one Group (guests)
- template/template.yaml: Node.js service scaffolder template
  - Steps: fetch:template -> publish:github -> catalog:register -> notification:send
  - Content: minimal index.js + package.json + catalog-info.yaml

## Key Design Decisions

- Using Backstage new backend system (not legacy)
- Allow-all permission policy (no RBAC yet)
- GitHub as the single integration and auth provider
- Development uses SQLite in-memory; production targets PostgreSQL
- Kubernetes plugin installed but not configured with any clusters

## What Needs to Be Built

- Java Spring Boot skeleton template (the core deliverable from brief.md)
- CI/CD pipeline configuration within the template (build, test, deploy)
- Potentially: Kubernetes cluster configuration, proper RBAC, production catalog sources
