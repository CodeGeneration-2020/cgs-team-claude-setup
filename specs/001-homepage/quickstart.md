# Quickstart: CFC Homepage

**Feature**: 001-homepage | **Date**: 2026-03-17

## Prerequisites

- Node.js (LTS)
- pnpm (package manager — npm and yarn are prohibited per constitution)
- PostgreSQL instance (local or Docker)
- Environment variables configured (see below)

## Environment Variables

```bash
# Database
APP_DATABASE_URL=postgresql://user:password@localhost:5432/cfc_dev

# AI Search Service (for semantic search embeddings)
APP_AI_SEARCH_API_KEY=<your-api-key>
APP_AI_SEARCH_MODEL=<embedding-model-name>

# General
APP_PORT=3000
APP_NODE_ENV=development
```

Copy `.env.example` to `.env` and fill in values. Never commit `.env`.

## Setup

```bash
# Install dependencies
pnpm install

# Generate Prisma client
pnpm --filter <backend-app> prisma generate

# Run database migrations
pnpm --filter <backend-app> prisma migrate dev

# Seed predefined taxonomy data (thematic areas, regions, sectors)
pnpm --filter <backend-app> prisma db seed
```

## Development

```bash
# Start all apps (backend + frontend) via Turborepo
pnpm dev

# Start backend only
pnpm --filter <backend-app> dev

# Start frontend only
pnpm --filter <web-app> dev
```

## Testing

```bash
# Run all tests
pnpm test

# Run backend unit tests for search module
pnpm --filter <backend-app> test -- --testPathPattern=search

# Run backend unit tests for trending module
pnpm --filter <backend-app> test -- --testPathPattern=trending

# Run frontend component tests for homepage
pnpm --filter <web-app> test -- --testPathPattern=homepage

# Run E2E tests for homepage
pnpm --filter <web-app> test:e2e -- --testPathPattern=homepage

# Run with coverage
pnpm test -- --coverage
```

## Key Development Notes

### Backend Modules to Implement
1. **Search Module** (`src/modules/search/`) — semantic search endpoint, filter logic, pagination
2. **Trending Module** (`src/modules/trending/`) — public trending list, admin CRUD

### Frontend Feature Modules to Implement
1. **Homepage** (`src/features/homepage/`) — search bar, trending section, results page, filters, sort
2. **Admin Trending** (`src/features/admin-trending/`) — trending management page (Super Admin only)

### Shared Packages to Update
1. **shared-types** — add search, trending, and project card type definitions
2. **ui-components** — add ProjectCard and ProjectTag components
3. **api-client** — add search and trending API client methods

### Database Changes
1. Add `trending_projects` table (new Prisma model)
2. Add vector embedding support for project search (pgvector or separate index)
3. Seed predefined taxonomy values

### API Endpoints
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /v1/search | Public | Semantic search with filters and sort |
| GET | /v1/search/filters | Public | Available filter options |
| GET | /v1/trending | Public | Current trending projects |
| GET | /v1/admin/trending | Super Admin | Admin view of trending list |
| PUT | /v1/admin/trending | Super Admin | Update trending list |
