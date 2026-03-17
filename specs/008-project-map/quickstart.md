# Quickstart: Project Map

**Feature**: 008-project-map | **Date**: 2026-03-17

## Prerequisites

- Feature 003-project-management implemented (project detail page exists)
- Geocoding service access (Nominatim/OSM is free, no API key needed)

## Environment Variables

```bash
# Optional — only if using Mapbox instead of OSM
APP_MAP_PROVIDER=osm                    # "osm" (default, free) or "mapbox"
APP_MAPBOX_ACCESS_TOKEN=<token>         # only needed if APP_MAP_PROVIDER=mapbox
```

## Setup

```bash
pnpm install
pnpm --filter <backend-app> prisma generate
pnpm --filter <backend-app> prisma migrate dev    # adds latitude, longitude, geocode_status to projects
```

## Testing

```bash
pnpm --filter packages/ui-components test -- --testPathPattern=ProjectMap
pnpm --filter <web-app> test -- --testPathPattern=project-map
pnpm --filter <web-app> test:e2e -- --testPathPattern=project-map
```

## Key Changes

- **Backend**: Add geocoding service call on project create/edit (async). Add lat/lng to project response DTO.
- **Frontend**: ProjectMap shared component (Leaflet + OSM tiles). ProjectLocationMap wrapper with geocode fallback. Integrated into project detail page.
- **No new API endpoints** — map data is included in existing GET /v1/projects/:id response.
