# Tech

## Core Framework
- Backstage v1.48.0 (backstage.json)
- New backend system (@backstage/backend-defaults createBackend)

## Languages & Runtime
- TypeScript ~5.8.0
- Node.js 22 or 24 (engines field in root package.json)
- React 18 (frontend SPA)
- JSX: react-jsx (tsconfig)

## Package Manager
- Yarn 4.4.1 (berry) with node-modules linker
- Workspaces: packages/* and plugins/*

## Frontend Stack
- @backstage/app-defaults ^1.7.5
- @backstage/core-app-api ^1.19.5
- @backstage/core-components ^0.18.7
- @material-ui/core ^4.12.2 (Material UI v4)
- react-router-dom ^6.30.2

## Frontend Plugins Installed
- plugin-api-docs
- plugin-catalog + catalog-graph + catalog-import + catalog-react
- plugin-kubernetes (frontend)
- plugin-notifications + plugin-signals
- plugin-org
- plugin-permission-react
- plugin-scaffolder
- plugin-search + search-react
- plugin-techdocs + techdocs-react + techdocs-module-addons-contrib
- plugin-user-settings

## Backend Plugins Installed
- plugin-app-backend
- plugin-proxy-backend
- plugin-scaffolder-backend + module-github + module-notifications
- plugin-techdocs-backend
- plugin-auth-backend + module-github-provider + module-guest-provider
- plugin-catalog-backend + module-scaffolder-entity-model + module-logs
- plugin-permission-backend + module-allow-all-policy
- plugin-search-backend + module-pg + module-catalog + module-techdocs
- plugin-kubernetes-backend
- plugin-notifications-backend + plugin-signals-backend

## Database
- Development: better-sqlite3 (in-memory)
- Production: PostgreSQL (pg ^8.11.3)

## Authentication
- GitHub OAuth (dev clientId/clientSecret in app-config.yaml)
- Guest provider enabled for development

## Integration
- GitHub via Personal Access Token (GITHUB_TOKEN env var)

## Testing
- Jest ^30.2.0 (unit tests)
- Playwright ^1.32.3 (e2e tests)
- @testing-library/react ^14.0.0

## Build & Lint
- backstage-cli (build, lint, test commands)
- ESLint (.eslintrc.js extends backstage cli config)
- Prettier ^2.3.2

## Containerization
- Dockerfile in packages/backend
- Base image: node:24-trixie-slim
- Multi-step: skeleton install -> bundle extract -> run

## Development Setup
1. Clone repo
2. `yarn install`
3. Set GITHUB_TOKEN env var
4. `yarn start` (starts both frontend on :3000 and backend on :7007)

## Technical Constraints
- Kubernetes plugin present but no clusters configured
- CI/CD tab in EntityPage is a placeholder (EmptyState)
- TechDocs uses docker generator locally -- requires Docker running
- No custom Backstage plugins created yet (plugins/ dir is empty)
- GitHub OAuth clientId/clientSecret are hardcoded for dev only
