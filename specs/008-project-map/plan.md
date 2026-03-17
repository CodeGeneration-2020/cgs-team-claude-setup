# Implementation Plan: Project Map

**Branch**: `008-project-map` | **Date**: 2026-03-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/008-project-map/spec.md`

## Summary

Add a small, read-only location map to the project detail page. The map geocodes the project's location text into coordinates, renders a map tile with a pin or region highlight, and degrades gracefully when data is missing or the service is unavailable. No new backend endpoints — geocoding and map rendering happen on the frontend using a map library and geocoding service.

## Technical Context

**Language/Version**: TypeScript (strict mode)
**Primary Dependencies**: NestJS (backend — minimal changes), React.js + Tailwind CSS (frontend), map library (Leaflet or Mapbox GL JS), geocoding service
**Storage**: PostgreSQL — optional latitude/longitude cache columns on project
**Testing**: Jest (unit), React Testing Library (components), Playwright (E2E)
**Target Platform**: Web (desktop + mobile responsive)
**Project Type**: Web service (monorepo)
**Performance Goals**: Map loads within 2 seconds of page load
**Constraints**: Map must not dominate content, max 25% viewport on desktop / 30% on mobile, graceful fallback
**Scale/Scope**: One map per project detail page, no global map

## Constitution Check

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Monorepo-First | PASS | Adds component to existing web app |
| II. Clean Architecture & SOLID | PASS | Map is a pure UI component, geocoding in a service hook |
| III. Modular Architecture | PASS | Map component in packages/ui-components or within existing project detail feature |
| IV. Strict Type Safety | PASS | Geocoding types added to shared-types |
| V. Security by Design | PASS | Map API key via env var, no secrets in code |
| VI. Testing Discipline | PASS | Component tests for map rendering and fallback |
| VII. Independent Deployability | PASS | No new apps |
| VIII. Observability-First | PASS | Geocoding errors logged |
| IX. Shared-Before-Custom | PASS | ProjectMap as shared component in ui-components |
| X. Design Token Management | PASS | Map container sizing via design tokens |

**Gate Result: PASS**

## Project Structure

### Documentation

```text
specs/008-project-map/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code

```text
# Shared UI component
packages/ui-components/src/
├── ProjectMap/
│   ├── ProjectMap.tsx
│   ├── ProjectMap.stories.tsx
│   └── ProjectMap.spec.tsx

# Frontend hook for geocoding
apps/<web-app>/src/features/project-detail/
├── hooks/
│   └── useGeocode.ts          # geocodes location text → lat/lng
├── ui/
│   └── components/
│       └── ProjectLocationMap.tsx   # wraps ProjectMap with geocoding logic

# Optional: backend geocoding cache
apps/<backend-app>/src/modules/projects/
├── dto/
│   └── project-response.dto.ts    # extended with optional lat/lng

# E2E
tests/e2e/project-map/
└── project-map.spec.ts
```

**Structure Decision**: ProjectMap as a shared UI component (Constitution IX) since it may be reused. Geocoding happens client-side via a hook. Optional server-side latitude/longitude cache to avoid re-geocoding on every page load. Minimal backend changes.

## Complexity Tracking

No violations.
