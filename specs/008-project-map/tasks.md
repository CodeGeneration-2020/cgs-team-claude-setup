# Tasks: Project Map

**Input**: Design documents from `/specs/008-project-map/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, quickstart.md

**Tests**: TDD — constitution requires 80% coverage.

**Organization**: Single user story (US1) — small focused feature.

## Format: `[ID] [P?] [Story] Description`

## Path Conventions

- **Backend**: `apps/<backend-app>/src/modules/`
- **Frontend**: `apps/<web-app>/src/features/`
- **UI components**: `packages/ui-components/src/`
- **E2E tests**: `tests/e2e/project-map/`

---

## Phase 1: Setup

- [ ] T001 Create Prisma migration to add latitude (Float nullable), longitude (Float nullable), geocode_status (String default "pending") to projects table

---

## Phase 2: Foundational

- [ ] T002 Install Leaflet and react-leaflet (or equivalent) as dependencies in packages/ui-components and apps/<web-app>
- [ ] T003 [P] Add latitude, longitude, geocodeStatus fields to the existing project response DTO in packages/shared-types/src/project/project-response.ts

---

## Phase 3: US1 — Project Location Map (Priority: P1)

**Goal**: Project detail page shows a small Leaflet/OSM map with a pin on the project's location. Map hidden when no coordinates. Graceful fallback on errors.

**Independent Test**: Open a project with location "Kenya, East Africa" → see map with pin centered on Kenya → resize to mobile → map still visible and proportionate.

### Backend — Geocoding

- [ ] T004 [US1] Create geocoding service in apps/<backend-app>/src/modules/projects/services/geocoding.service.ts — accepts location text, calls Nominatim/OSM geocoding API, returns { latitude, longitude } or null on failure
- [ ] T005 [US1] Integrate geocoding into ProjectsService.create() and ProjectsService.update() — after save, asynchronously geocode the location field, store lat/lng and geocode_status on the project record. Don't block the save response
- [ ] T006 [US1] Add latitude and longitude to GET /v1/projects/:id response in ProjectResponseDto
- [ ] T007 [US1] Write unit tests for GeocodingService in apps/<backend-app>/src/modules/projects/__tests__/geocoding.spec.ts — valid location, broad region, unmappable text, API failure

### Frontend — Map Component

- [ ] T008 [US1] Create ProjectMap shared component in packages/ui-components/src/ProjectMap/ProjectMap.tsx — accepts latitude, longitude, zoom props. Renders Leaflet map with OSM tiles and a pin marker. Fixed height, responsive width. Design token sizing
- [ ] T009 [P] [US1] Create ProjectMap.stories.tsx in packages/ui-components/src/ProjectMap/ — stories for: city pin, country zoom, loading
- [ ] T010 [P] [US1] Create ProjectMap.spec.tsx in packages/ui-components/src/ProjectMap/ — renders map with valid coords, hidden when coords null
- [ ] T011 [US1] Create ProjectLocationMap wrapper in apps/<web-app>/src/features/project-detail/ui/components/ProjectLocationMap.tsx — reads project lat/lng from detail data, renders ProjectMap if coordinates exist, renders nothing if null. Shows "Map unavailable" placeholder on Leaflet load error
- [ ] T012 [US1] Integrate ProjectLocationMap into the project detail page — positioned below description or in a sidebar, max 25% viewport height on desktop, 30% on mobile. Does not block main content
- [ ] T013 [US1] Write component test for ProjectLocationMap in apps/<web-app>/src/features/project-detail/__tests__/ProjectLocationMap.spec.tsx — with coords, without coords (hidden), error fallback

---

## Phase 4: Polish

- [ ] T014 [P] Verify responsive map sizing — desktop (max 25% viewport), tablet, mobile (max 30% viewport)
- [ ] T015 [P] Write E2E test for project map in tests/e2e/project-map/project-map.spec.ts — project with location shows map, project without location has no map
- [ ] T016 Run quickstart.md validation

---

## Dependencies & Execution Order

```
Phase 1 (Migration)
    │
Phase 2 (Dependencies + types)
    │
Phase 3 (US1: Geocoding + Map component + Integration)
    │
Phase 4 (Polish + E2E)
```

### Parallel Opportunities

- T002+T003 in parallel
- T008+T009+T010 (component + stories + tests) in parallel
- T014+T015 (responsive + E2E) in parallel

---

## Implementation Strategy

1. Phase 1+2: Migration + deps (quick)
2. Phase 3: Geocoding service + map component + integration
3. **STOP and VALIDATE**: Open a project detail page, verify map shows
4. Phase 4: Polish + E2E

**Estimated scope**: Small feature, ~16 tasks, 1-2 dev days.

---

## Notes

- Leaflet + OSM = free, no API key needed for tiles
- Geocoding via Nominatim (OSM) — free, no key needed, rate-limited (1 req/sec). Acceptable since geocoding only on create/edit, not on every page view
- Pin marker for all locations — no GeoJSON region polygons in v1
- Zoom level determined by geocoding precision: city=12, country=6, region=4
- Map is purely read-only — no zoom/pan controls exposed to user
