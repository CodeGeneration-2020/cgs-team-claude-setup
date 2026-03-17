# Tasks: CFC Homepage

**Input**: Design documents from `/specs/001-homepage/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: TDD approach — constitution requires 80% coverage and tests before implementation.

**Organization**: Tasks grouped by user story for independent implementation and testing.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2)
- Paths use monorepo structure from plan.md

## Path Conventions

- **Backend**: `apps/<backend-app>/src/modules/`
- **Frontend**: `apps/<web-app>/src/features/`
- **Shared types**: `packages/shared-types/src/`
- **UI components**: `packages/ui-components/src/`
- **API client**: `packages/api-client/src/`
- **E2E tests**: `tests/e2e/homepage/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization, shared types, and base structure

- [ ] T001 Add search and trending shared type definitions in packages/shared-types/src/search/search-query.ts, search-result.ts, search-filters.ts, index.ts
- [ ] T002 [P] Add project card and tag shared type definitions in packages/shared-types/src/project/project-card.ts, project-status.ts, project-tag.ts, index.ts
- [ ] T003 [P] Add trending project shared type definitions in packages/shared-types/src/trending/trending-project.ts, index.ts
- [ ] T004 Add SearchSortOption and ProjectStatus enum definitions in packages/shared-types/src/search/search-sort-option.ts and packages/shared-types/src/project/project-status.ts
- [ ] T005 Re-export all new types from packages/shared-types/src/index.ts barrel

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Database schema, search infrastructure, and shared UI components that all stories depend on

**CRITICAL**: No user story work can begin until this phase is complete

- [ ] T006 Create Prisma migration for trending_projects table with fields: id, project_id (unique FK), display_order, added_by (FK to user), created_at, updated_at. Add indexes idx_trending_projects_display_order and idx_trending_projects_project_id
- [ ] T007 [P] Create Prisma migration to add vector embedding support for project semantic search (pgvector extension or embedding column on projects table)
- [ ] T008 [P] Create seed script for predefined taxonomy values (thematic areas, regions, stakeholder sectors, lifecycle statuses) in the database
- [ ] T009 [P] Create ProjectCard component in packages/ui-components/src/ProjectCard/ProjectCard.tsx — accepts ProjectCardData type, displays title, organization, thematic area, location, status, funding, short description, tags. Responsive (full-width mobile, grid desktop)
- [ ] T010 [P] Create ProjectCard.stories.tsx and ProjectCard.spec.tsx in packages/ui-components/src/ProjectCard/
- [ ] T011 [P] Create ProjectTag component in packages/ui-components/src/ProjectTag/ProjectTag.tsx — renders color-coded pill-shaped labels for thematic area and status tags. Add ProjectTag.stories.tsx and ProjectTag.spec.tsx
- [ ] T012 Add search and trending API client methods in packages/api-client/src/ — postSearch(), getSearchFilters(), getTrending(), getAdminTrending(), updateAdminTrending()

**Checkpoint**: Foundation ready — shared types, database schema, UI components, and API client all in place

---

## Phase 3: US1 + US2 + US3 — Search on Homepage + AI Matching + Results Display (Priority: P1) MVP

**Goal**: Visitor can type a natural-language query on the homepage, receive AI-powered semantic search results in a structured card layout, and navigate to project detail pages. Only Active/Approved projects appear.

**Independent Test**: Open homepage → type query → see card results → click to view project detail. Verify semantic matching (different wording, same results).

### Backend — Search Module

- [ ] T013 [US1] Create search module scaffold: apps/<backend-app>/src/modules/search/search.module.ts, search.controller.ts, search.service.ts, dto/, entities/, index.ts
- [ ] T014 [P] [US1] Create SearchQueryDto in apps/<backend-app>/src/modules/search/dto/search-query.dto.ts — validates query (required, max 500 chars, sanitized), filters (optional), sort (enum), cursor (optional), pageSize (default 12, max 50)
- [ ] T015 [P] [US1] Create SearchResultDto and SearchResponseDto in apps/<backend-app>/src/modules/search/dto/search-results.dto.ts — maps to contracts/search.md response shape including meta with totalResults, nextCursor, hasMore, isRefinementSuggested
- [ ] T016 [US2] Implement SearchService.search() in apps/<backend-app>/src/modules/search/search.service.ts — generate embedding for query, run vector similarity search against projects, filter to Active/Approved status, apply cursor pagination, return ranked results with relevance scores
- [ ] T017 [US2] Implement ambiguity detection in SearchService — if query < 3 meaningful words or top results have low similarity scores, set isRefinementSuggested=true with helpful message
- [ ] T018 [US1] Implement SearchController.search() in apps/<backend-app>/src/modules/search/search.controller.ts — POST /v1/search, delegates to service, returns standard response envelope { success, data, meta }
- [ ] T019 [US1] Add input sanitization middleware for search queries — strip HTML tags, normalize whitespace, enforce 500-char limit
- [ ] T020 [US1] Add structured logging for search queries in SearchService — log query text, result count, duration, filters applied (no PII)
- [ ] T021 [US1] Write unit tests for SearchService in apps/<backend-app>/src/modules/search/__tests__/search.service.spec.ts — test semantic search, status filtering, pagination, ambiguity detection, empty results
- [ ] T022 [P] [US1] Write unit tests for SearchController in apps/<backend-app>/src/modules/search/__tests__/search.controller.spec.ts — test request validation, response shape, error handling
- [ ] T023 [US1] Write integration test for POST /v1/search endpoint in apps/<backend-app>/src/modules/search/__tests__/search.integration.spec.ts

### Frontend — Homepage Search + Results

- [ ] T024 [US1] Create homepage feature module scaffold: apps/<web-app>/src/features/homepage/ with ui/pages/, ui/components/, state/, services/, hooks/, domain/, index.ts
- [ ] T025 [US1] Create SearchBar component in apps/<web-app>/src/features/homepage/ui/components/SearchBar.tsx — prominent search input, submit button, visible above fold
- [ ] T026 [US1] Create useSearch hook in apps/<web-app>/src/features/homepage/hooks/useSearch.ts — manages search query submission via TanStack Query, calls api-client postSearch()
- [ ] T027 [US3] Create SearchResults component in apps/<web-app>/src/features/homepage/ui/components/SearchResults.tsx — renders list of ProjectCard components from search results, handles loading/empty/error states
- [ ] T028 [US1] Create HomePage in apps/<web-app>/src/features/homepage/ui/pages/HomePage.tsx — layout with SearchBar above fold, SearchResults below. Route: /
- [ ] T029 [US3] Implement "no results" and "refinement suggested" states in SearchResults — display appropriate messages per US2/US3 acceptance criteria
- [ ] T030 [US1] Implement cursor-based pagination in SearchResults — "Load more" button or infinite scroll, calls next page via cursor
- [ ] T031 [US1] Write component tests for SearchBar in apps/<web-app>/src/features/homepage/__tests__/SearchBar.spec.tsx
- [ ] T032 [P] [US3] Write component tests for SearchResults in apps/<web-app>/src/features/homepage/__tests__/SearchResults.spec.tsx

**Checkpoint**: US1+US2+US3 complete — visitor can search from homepage, get AI-matched results in cards, paginate, see empty/refinement states

---

## Phase 4: US4 — Filter Search Results (Priority: P2)

**Goal**: Visitor can apply filters (thematic area, region, lifecycle, organization, funding range, stakeholder sector, Local NGO) to narrow AI search results without re-running the search.

**Independent Test**: Run a search → apply filters → verify results narrow → remove filters → verify original results restored.

### Backend — Filter Support

- [ ] T033 [US4] Create SearchFiltersDto in apps/<backend-app>/src/modules/search/dto/search-filters.dto.ts — validates all filter fields: thematicArea (string[]), region (string[]), lifecycleStatus (string[]), organization (string[]), fundingMin/fundingMax (number), stakeholderSector (string[]), isLocalNgo (boolean)
- [ ] T034 [US4] Implement filter application in SearchService.search() — apply SQL WHERE clauses on the AI search result set for each active filter. Filters narrow results without re-running the embedding search
- [ ] T035 [US4] Implement GET /v1/search/filters endpoint in SearchController — returns distinct values for all filter dimensions from active project data
- [ ] T036 [US4] Write unit tests for filter logic in apps/<backend-app>/src/modules/search/__tests__/search-filters.spec.ts — test individual filters, combined filters, empty filter results, filter options endpoint

### Frontend — Filter Panel

- [ ] T037 [US4] Create FilterPanel component in apps/<web-app>/src/features/homepage/ui/components/FilterPanel.tsx — renders filter controls for all 7 dimensions, supports multi-select, range input for funding, checkbox for Local NGO
- [ ] T038 [US4] Create useFilters hook in apps/<web-app>/src/features/homepage/hooks/useFilters.ts — manages filter state, populates options from GET /v1/search/filters, applies/removes/clears filters
- [ ] T039 [US4] Integrate FilterPanel into SearchResults page — filters visible alongside results, applying/removing filters updates results via search API with filter params
- [ ] T040 [US4] Implement "Clear all filters" button and individual filter removal in FilterPanel
- [ ] T041 [US4] Implement "no results with current filters" message when filter combination returns zero results
- [ ] T042 [US4] Write component tests for FilterPanel in apps/<web-app>/src/features/homepage/__tests__/FilterPanel.spec.tsx

**Checkpoint**: US4 complete — filters narrow search results, can be combined, individually removed, or cleared entirely

---

## Phase 5: US5 — Sort Search Results (Priority: P2)

**Goal**: Visitor can sort results by relevance, date (newest/oldest), or name without removing filters or re-running the AI search.

**Independent Test**: Run a search → change sort → verify order changes → apply filter → change sort → verify filters preserved.

- [ ] T043 [US5] Implement sort parameter handling in SearchService.search() — apply ORDER BY clause based on sort option (relevance score, created_at ASC/DESC, title ASC). Sorting does not re-run embedding search
- [ ] T044 [US5] Create SortControls component in apps/<web-app>/src/features/homepage/ui/components/SortControls.tsx — dropdown or button group for sort options: Relevance, Newest, Oldest, Name A-Z
- [ ] T045 [US5] Integrate SortControls into SearchResults page — sort changes trigger re-fetch with sort param, preserves active filters
- [ ] T046 [US5] Write unit test for sort logic in apps/<backend-app>/src/modules/search/__tests__/search-sort.spec.ts
- [ ] T047 [P] [US5] Write component test for SortControls in apps/<web-app>/src/features/homepage/__tests__/SortControls.spec.tsx

**Checkpoint**: US5 complete — results can be reordered by relevance, date, or name without losing filters

---

## Phase 6: US6 — Trending Projects Section (Priority: P2)

**Goal**: Homepage displays a curated Trending Projects section showing featured projects selected by Super Admin. Each card shows project title, org, thematic area, status, funding, location, summary.

**Independent Test**: Open homepage → see trending section → verify project cards → click card → navigate to project detail.

### Backend — Trending Public Endpoint

- [ ] T048 [US6] Create trending module scaffold: apps/<backend-app>/src/modules/trending/trending.module.ts, trending.controller.ts, trending.service.ts, dto/, entities/, index.ts
- [ ] T049 [US6] Create TrendingProject entity in apps/<backend-app>/src/modules/trending/entities/trending-project.entity.ts — maps to trending_projects table from data-model.md
- [ ] T050 [US6] Create TrendingProjectDto in apps/<backend-app>/src/modules/trending/dto/trending-project.dto.ts — response shape per contracts/trending.md
- [ ] T051 [US6] Implement TrendingService.getPublicTrending() in apps/<backend-app>/src/modules/trending/trending.service.ts — return trending projects ordered by display_order, join with project data, exclude non-Active/Approved projects
- [ ] T052 [US6] Implement GET /v1/trending in TrendingController — public endpoint, returns ordered trending projects
- [ ] T053 [US6] Write unit tests for TrendingService in apps/<backend-app>/src/modules/trending/__tests__/trending.service.spec.ts
- [ ] T054 [P] [US6] Write integration test for GET /v1/trending in apps/<backend-app>/src/modules/trending/__tests__/trending.integration.spec.ts

### Frontend — Trending Section

- [ ] T055 [US6] Create TrendingSection component in apps/<web-app>/src/features/homepage/ui/components/TrendingSection.tsx — displays trending project cards, visually distinct from search results, handles empty state
- [ ] T056 [US6] Create useTrending hook in apps/<web-app>/src/features/homepage/hooks/useTrending.ts — fetches trending projects via TanStack Query
- [ ] T057 [US6] Integrate TrendingSection into HomePage — display alongside search bar, visible on initial page load before any search is performed
- [ ] T058 [US6] Write component test for TrendingSection in apps/<web-app>/src/features/homepage/__tests__/TrendingSection.spec.tsx

**Checkpoint**: US6 complete — homepage shows trending projects, cards link to detail pages, empty state handled

---

## Phase 7: US7 — Visual Tags on Projects (Priority: P2)

**Goal**: Project cards display color-coded visual tags for thematic area and status. Tags appear on both cards and project detail pages.

**Independent Test**: View search results → verify tags on cards → open project detail → verify same tags displayed.

- [ ] T059 [US7] Integrate ProjectTag component into ProjectCard — render thematic area and status tags on every card in search results and trending section
- [ ] T060 [US7] Add tag rendering to the project detail page — display same tags as shown on cards
- [ ] T061 [US7] Define tag color mapping in packages/ui-components/src/ProjectTag/tag-colors.ts — map thematic areas and statuses to design token colors (no hardcoded hex values per constitution Principle X)
- [ ] T062 [US7] Write visual regression test for ProjectTag and ProjectCard with tags in packages/ui-components/src/ProjectTag/ProjectTag.spec.tsx

**Checkpoint**: US7 complete — tags visible on all project cards and detail pages, consistent colors from design tokens

---

## Phase 8: US8 — Manage Trending Projects — Super Admin (Priority: P3)

**Goal**: Super Admin can add, remove, reorder trending projects through an admin interface. Changes reflect on homepage immediately.

**Independent Test**: Login as Super Admin → open trending management → add project → reorder → save → verify homepage updates.

### Backend — Admin Trending Endpoints

- [ ] T063 [US8] Create UpdateTrendingDto in apps/<backend-app>/src/modules/trending/dto/update-trending.dto.ts — validates projects array with projectId and displayOrder fields
- [ ] T064 [US8] Implement TrendingService.getAdminTrending() — return full trending list including inactive projects with projectStatus field
- [ ] T065 [US8] Implement TrendingService.updateTrending() — replace entire trending list atomically, validate all projectIds exist
- [ ] T066 [US8] Implement GET /v1/admin/trending in TrendingController — protected by Auth Guard + Super Admin Role Guard, returns admin view
- [ ] T067 [US8] Implement PUT /v1/admin/trending in TrendingController — protected by Auth Guard + Super Admin Role Guard, replaces trending list
- [ ] T068 [US8] Write unit tests for admin trending operations in apps/<backend-app>/src/modules/trending/__tests__/trending-admin.spec.ts
- [ ] T069 [P] [US8] Write integration tests for admin trending endpoints in apps/<backend-app>/src/modules/trending/__tests__/trending-admin.integration.spec.ts — test auth, role guard, CRUD, invalid project IDs

### Frontend — Admin Trending Management

- [ ] T070 [US8] Create admin-trending feature module scaffold: apps/<web-app>/src/features/admin-trending/ with ui/pages/, ui/components/, state/, services/, hooks/, index.ts
- [ ] T071 [US8] Create ManageTrendingPage in apps/<web-app>/src/features/admin-trending/ui/pages/ManageTrendingPage.tsx — route: /admin/trending, requires Super Admin role
- [ ] T072 [US8] Create TrendingList component in apps/<web-app>/src/features/admin-trending/ui/components/TrendingList.tsx — displays current trending projects with reorder controls (drag-and-drop or up/down buttons) and remove button
- [ ] T073 [US8] Create AddProjectDialog component in apps/<web-app>/src/features/admin-trending/ui/components/AddProjectDialog.tsx — search and select a project to add to trending
- [ ] T074 [US8] Create useTrendingAdmin hook in apps/<web-app>/src/features/admin-trending/hooks/useTrendingAdmin.ts — fetches admin trending list, handles add/remove/reorder/save via API client
- [ ] T075 [US8] Implement save functionality — PUT /v1/admin/trending with current list, show success/error feedback
- [ ] T076 [US8] Write component tests for ManageTrendingPage, TrendingList, AddProjectDialog in apps/<web-app>/src/features/admin-trending/__tests__/

**Checkpoint**: US8 complete — Super Admin can manage trending list, changes immediately visible on homepage

---

## Phase 9: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T077 [P] Add responsive layout adjustments for homepage — verify search bar, trending, results, filters all work on mobile (375px), tablet (768px), desktop (1440px)
- [ ] T078 [P] Add loading skeletons for search results and trending section during data fetching
- [ ] T079 Add keyboard navigation support for search bar submit (Enter key), filter controls, and sort controls
- [ ] T080 [P] Add SearchLog entity and logging in SearchService — persist anonymized search analytics per data-model.md SearchLog entity
- [ ] T081 [P] Add error boundary for homepage — graceful fallback when AI search service is unavailable (show trending projects and suggest browsing by category)
- [ ] T082 Run quickstart.md validation — verify all setup steps, dev commands, and test commands work end-to-end
- [ ] T083 [P] Write E2E tests in tests/e2e/homepage/search.spec.ts — homepage search flow, result display, pagination
- [ ] T084 [P] Write E2E tests in tests/e2e/homepage/filters.spec.ts — filter application, combination, clear
- [ ] T085 [P] Write E2E tests in tests/e2e/homepage/trending.spec.ts — trending section display, card navigation
- [ ] T086 [P] Write E2E tests in tests/e2e/homepage/sort.spec.ts — sort options, order verification

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies — start immediately
- **Phase 2 (Foundational)**: Depends on Phase 1 — BLOCKS all user stories
- **Phase 3 (US1+US2+US3)**: Depends on Phase 2 — MVP search experience
- **Phase 4 (US4)**: Depends on Phase 3 (extends search results with filters)
- **Phase 5 (US5)**: Depends on Phase 3 (extends search results with sorting). Can run in parallel with Phase 4
- **Phase 6 (US6)**: Depends on Phase 2 only — can run in parallel with Phase 3
- **Phase 7 (US7)**: Depends on Phase 2 (uses shared ProjectTag component). Can run in parallel with Phase 3
- **Phase 8 (US8)**: Depends on Phase 6 (extends trending with admin management)
- **Phase 9 (Polish)**: Depends on all desired user stories being complete

### User Story Dependencies

```
Phase 1 (Setup)
    │
Phase 2 (Foundation)
    │
    ├──────────────┬──────────────┬──────────────┐
    │              │              │              │
Phase 3         Phase 6       Phase 7         (parallel)
US1+US2+US3     US6 Trending  US7 Tags
    │              │
    ├──────┐       │
    │      │       │
Phase 4  Phase 5  Phase 8
US4      US5      US8 Admin
Filters  Sort     Trending
    │      │       │
    └──────┴───────┘
           │
       Phase 9
       Polish
```

### Within Each User Story

- Models/entities before services
- Services before controllers/endpoints
- Backend before frontend integration
- Core implementation before edge cases
- Unit tests alongside implementation

### Parallel Opportunities

- **Phase 1**: T001-T005 — shared type definitions can all run in parallel
- **Phase 2**: T006-T012 — DB migrations, seed script, UI components, API client in parallel
- **Phase 3**: Backend (T013-T023) and frontend (T024-T032) can partially overlap once DTOs are defined
- **Phase 4+5**: Can run in parallel with each other (both extend Phase 3 results)
- **Phase 6+7**: Can run in parallel with Phase 3 (independent of search results)
- **Phase 9**: All E2E tests (T083-T086) can run in parallel

---

## Parallel Example: Phase 3 (US1+US2+US3)

```bash
# After T013 (module scaffold), launch in parallel:
Task T014: "Create SearchQueryDto in apps/<backend-app>/src/modules/search/dto/search-query.dto.ts"
Task T015: "Create SearchResultDto in apps/<backend-app>/src/modules/search/dto/search-results.dto.ts"

# After T016 (SearchService), launch in parallel:
Task T021: "Unit tests for SearchService"
Task T022: "Unit tests for SearchController"

# Frontend can start after DTOs are defined:
Task T024: "Create homepage feature module scaffold"
Task T025: "Create SearchBar component"
Task T027: "Create SearchResults component"
```

---

## Implementation Strategy

### MVP First (Phase 1 + 2 + 3 = US1+US2+US3)

1. Complete Phase 1: Shared types
2. Complete Phase 2: DB schema, UI components, API client
3. Complete Phase 3: Search on homepage with AI matching and card results
4. **STOP and VALIDATE**: Visitor can search, see AI-matched results, paginate
5. Deploy/demo MVP

### Incremental Delivery

1. **Phase 1+2+3** → Search works end-to-end (MVP)
2. **Phase 4** → Add filters → Deploy
3. **Phase 5** → Add sorting → Deploy
4. **Phase 6** → Add trending section → Deploy
5. **Phase 7** → Add visual tags → Deploy
6. **Phase 8** → Add admin trending management → Deploy
7. **Phase 9** → Polish, E2E tests, responsive fixes → Final release

### Parallel Team Strategy

With 2 developers after Phase 2:
- **Dev A**: Phase 3 (search) → Phase 4 (filters) → Phase 5 (sort)
- **Dev B**: Phase 6 (trending) → Phase 7 (tags) → Phase 8 (admin trending)
- **Together**: Phase 9 (polish + E2E)

---

## Notes

- [P] tasks = different files, no dependencies on incomplete tasks
- [Story] label maps task to specific user story for traceability
- US1+US2+US3 are grouped because they form the core search experience — inseparable for MVP
- Constitution requires 80% test coverage — tests included alongside implementation
- All UI colors use design tokens (Principle X), no hardcoded hex values
- All shared types go through packages/shared-types barrel exports (Principle IV)
- Admin endpoints protected by Auth Guard + Role Guard (Principle V)
