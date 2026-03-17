# Tasks: Bookmarks

**Input**: Design documents from `/specs/004-bookmarks/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: TDD — constitution requires 80% coverage.

**Organization**: US1 (Bookmark Toggle) and US2 (View Bookmarks) grouped as P1 MVP — toggle is useless without a list to view them. US3 (Track Updates) is P2.

## Format: `[ID] [P?] [Story] Description`

## Path Conventions

- **Backend**: `apps/<backend-app>/src/modules/`
- **Frontend**: `apps/<web-app>/src/features/`
- **Shared types**: `packages/shared-types/src/`
- **UI components**: `packages/ui-components/src/`
- **API client**: `packages/api-client/src/`
- **E2E tests**: `tests/e2e/bookmarks/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Shared types and API client

- [ ] T001 Add bookmark types in packages/shared-types/src/bookmark/bookmark.ts — Bookmark response shape with bookmarkId, project, isAvailable, hasNewUpdates, bookmarkedAt
- [ ] T002 [P] Add bookmark update type in packages/shared-types/src/bookmark/bookmark-update.ts — projectId, projectTitle, changeType, changeSummary, occurredAt, isNew
- [ ] T003 Re-export from packages/shared-types/src/bookmark/index.ts and packages/shared-types/src/index.ts

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Database schema, shared UI component, API client

- [ ] T004 Create Prisma migration for bookmarks table: id (CUID), user_id (FK to users), project_id (FK to projects, SET NULL on delete, nullable), last_seen_at (DateTime), created_at. Add unique index on (user_id, project_id), indexes on user_id and project_id
- [ ] T005 [P] Create BookmarkButton component in packages/ui-components/src/BookmarkButton/BookmarkButton.tsx — accepts isBookmarked, onToggle, isLoading props. Renders heart/bookmark icon with filled/outline states. Add BookmarkButton.stories.tsx and BookmarkButton.spec.tsx
- [ ] T006 [P] Add bookmark API client methods in packages/api-client/src/bookmarks/ — addBookmark(), removeBookmark(), getBookmarks(), getBookmarkUpdates(), markBookmarkSeen(), checkBookmarks()
- [ ] T007 Integrate BookmarkButton into existing ProjectCard component in packages/ui-components/src/ProjectCard/ProjectCard.tsx — add optional onBookmarkToggle and isBookmarked props

**Checkpoint**: Foundation ready — DB schema, shared button, API client

---

## Phase 3: US1 + US2 — Bookmark Toggle + View My Bookmarks (Priority: P1) MVP

**Goal**: Funder can bookmark/un-bookmark projects from cards and detail pages. Funder can view all bookmarks in a dedicated "My Bookmarks" page with pagination. Stale bookmarks (deleted projects) marked as unavailable.

**Independent Test**: Log in as Funder → search for project → click bookmark icon → icon fills → navigate to My Bookmarks → see project listed → click bookmark icon again → project removed.

### Backend — Bookmarks Module

- [ ] T008 [US1] Create bookmarks module scaffold: apps/<backend-app>/src/modules/bookmarks/bookmarks.module.ts, bookmarks.controller.ts, bookmarks.service.ts, dto/, entities/, index.ts
- [ ] T009 [US1] Create Bookmark entity in apps/<backend-app>/src/modules/bookmarks/entities/bookmark.entity.ts — maps to bookmarks table
- [ ] T010 [P] [US2] Create BookmarksListQueryDto in apps/<backend-app>/src/modules/bookmarks/dto/bookmarks-list-query.dto.ts — page, pageSize validation
- [ ] T011 [P] [US1] Create BookmarkResponseDto in apps/<backend-app>/src/modules/bookmarks/dto/bookmark-response.dto.ts — response shape per contracts/bookmarks.md
- [ ] T012 [US1] Implement BookmarksService.addBookmark() in apps/<backend-app>/src/modules/bookmarks/bookmarks.service.ts — check project exists, upsert bookmark (idempotent), return bookmark data
- [ ] T013 [US1] Implement BookmarksService.removeBookmark() — delete bookmark if exists (idempotent), return success
- [ ] T014 [US2] Implement BookmarksService.getBookmarks() — query user's bookmarks with LEFT JOIN to projects, pagination, include isAvailable flag for deleted projects, hasNewUpdates flag based on last_seen_at vs audit log
- [ ] T015 [US1] Implement BookmarksService.checkBookmarks() — given list of project IDs, return map of projectId → isBookmarked for the current user
- [ ] T016 [US1] Implement POST /v1/bookmarks/:projectId in BookmarksController — @Roles(FUNDER, SUPER_ADMIN)
- [ ] T017 [US1] Implement DELETE /v1/bookmarks/:projectId in BookmarksController — @Roles(FUNDER, SUPER_ADMIN)
- [ ] T018 [US2] Implement GET /v1/bookmarks in BookmarksController — @Roles(FUNDER, SUPER_ADMIN), paginated
- [ ] T019 [US1] Implement GET /v1/bookmarks/check in BookmarksController — @Roles(FUNDER, SUPER_ADMIN), accepts projectIds query param
- [ ] T020 [US1] Write unit tests for BookmarksService in apps/<backend-app>/src/modules/bookmarks/__tests__/bookmarks.service.spec.ts — add, remove, idempotency, check, list with stale bookmarks
- [ ] T021 [P] [US2] Write integration tests for bookmark endpoints in apps/<backend-app>/src/modules/bookmarks/__tests__/bookmarks.integration.spec.ts — full CRUD flow, RBAC (Funder allowed, Org Admin blocked)

### Frontend — Bookmarks Feature Module

- [ ] T022 [US1] Create bookmarks feature module scaffold: apps/<web-app>/src/features/bookmarks/ with ui/pages/, ui/components/, state/, hooks/, index.ts
- [ ] T023 [US1] Create useBookmarkToggle hook in apps/<web-app>/src/features/bookmarks/hooks/useBookmarkToggle.ts — optimistic mutation via TanStack Query: toggle icon immediately, POST/DELETE in background, revert on error
- [ ] T024 [US1] Create useBookmarkStore (Zustand) in apps/<web-app>/src/features/bookmarks/state/useBookmarkStore.ts — caches bookmark states for visible project IDs, hydrated via checkBookmarks() on page load
- [ ] T025 [US1] Integrate BookmarkButton into search results — on search results page, call checkBookmarks() for visible project IDs, pass isBookmarked + onToggle to each ProjectCard
- [ ] T026 [P] [US1] Integrate BookmarkButton into project detail page — show bookmark toggle on detail page header
- [ ] T027 [US1] Integrate BookmarkButton into trending section — pass bookmark state to trending ProjectCards
- [ ] T028 [US1] Add login prompt when unauthenticated visitor clicks bookmark — redirect to login with return URL
- [ ] T029 [US2] Create useBookmarks hook in apps/<web-app>/src/features/bookmarks/hooks/useBookmarks.ts — fetches bookmarks list via TanStack Query with pagination
- [ ] T030 [US2] Create BookmarkList component in apps/<web-app>/src/features/bookmarks/ui/components/BookmarkList.tsx — renders bookmarked project cards with unbookmark action, "No longer available" for stale bookmarks, pagination
- [ ] T031 [US2] Create MyBookmarksPage in apps/<web-app>/src/features/bookmarks/ui/pages/MyBookmarksPage.tsx — route: /my-bookmarks (FUNDER only), renders BookmarkList, empty state with link to search
- [ ] T032 [US2] Add "My Bookmarks" link to Funder navigation
- [ ] T033 [US1] Write component tests for BookmarkButton integration in apps/<web-app>/src/features/bookmarks/__tests__/BookmarkToggle.spec.tsx — optimistic toggle, error revert, login prompt
- [ ] T034 [P] [US2] Write component tests for MyBookmarksPage in apps/<web-app>/src/features/bookmarks/__tests__/MyBookmarks.spec.tsx — list rendering, empty state, pagination, stale bookmarks

**Checkpoint**: US1+US2 complete — Funders can bookmark from anywhere, view all bookmarks in one place

---

## Phase 4: US3 — Track Updates on Bookmarked Projects (Priority: P2)

**Goal**: Funders see update indicators on bookmarked projects when significant changes occur (description, status, funding, stakeholders). Updates derived from project audit log.

**Independent Test**: Bookmark a project → project owner edits it → view My Bookmarks → see "new updates" indicator → click to view project → mark as seen → indicator clears.

### Backend

- [ ] T035 [US3] Implement BookmarksService.getUpdates() in apps/<backend-app>/src/modules/bookmarks/bookmarks.service.ts — query project_audit_log for bookmarked project IDs, filter to significant actions (EDITED with critical field changes, APPROVED, REJECTED), compare with last_seen_at, return BookmarkUpdate objects with isNew flag
- [ ] T036 [US3] Implement BookmarksService.markSeen() — update last_seen_at on the bookmark to current time
- [ ] T037 [US3] Implement GET /v1/bookmarks/updates in BookmarksController — @Roles(FUNDER, SUPER_ADMIN), paginated, optional onlyNew filter
- [ ] T038 [US3] Implement POST /v1/bookmarks/:projectId/mark-seen in BookmarksController — @Roles(FUNDER, SUPER_ADMIN)
- [ ] T039 [US3] Write unit tests for getUpdates and markSeen in apps/<backend-app>/src/modules/bookmarks/__tests__/bookmark-updates.spec.ts — update derivation from audit log, last_seen_at filtering, mark-seen

### Frontend

- [ ] T040 [US3] Create useBookmarkUpdates hook in apps/<web-app>/src/features/bookmarks/hooks/useBookmarkUpdates.ts — fetches updates via TanStack Query
- [ ] T041 [US3] Create BookmarkUpdateBadge component in apps/<web-app>/src/features/bookmarks/ui/components/BookmarkUpdateBadge.tsx — small indicator dot/badge on bookmarked projects with new updates
- [ ] T042 [US3] Add update indicators to MyBookmarksPage — show BookmarkUpdateBadge on projects with hasNewUpdates=true, show update summary on expand/click
- [ ] T043 [US3] Implement mark-as-seen — when Funder clicks through to a project from bookmarks, call mark-seen endpoint, clear the update indicator
- [ ] T044 [US3] Write component tests for bookmark updates in apps/<web-app>/src/features/bookmarks/__tests__/BookmarkUpdates.spec.tsx — badge rendering, update list, mark-seen

**Checkpoint**: US3 complete — Funders see updates on bookmarked projects, indicators clear after viewing

---

## Phase 5: Polish & Cross-Cutting Concerns

- [ ] T045 [P] Add responsive layout for My Bookmarks page — mobile 375px, tablet 768px, desktop 1440px
- [ ] T046 [P] Add loading skeleton for bookmark list during data fetching
- [ ] T047 [P] Write E2E test for bookmark toggle in tests/e2e/bookmarks/bookmark-toggle.spec.ts — bookmark from search, from detail page, from trending, persist across login
- [ ] T048 [P] Write E2E test for My Bookmarks page in tests/e2e/bookmarks/my-bookmarks.spec.ts — list, empty state, unbookmark, stale bookmarks
- [ ] T049 [P] Write E2E test for bookmark updates in tests/e2e/bookmarks/bookmark-updates.spec.ts — update indicator after project change, mark-seen
- [ ] T050 Run quickstart.md validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies
- **Phase 2 (Foundation)**: Depends on Phase 1
- **Phase 3 (US1+US2)**: Depends on Phase 2 — MVP bookmark + list
- **Phase 4 (US3)**: Depends on Phase 3 (extends bookmarks with updates)
- **Phase 5 (Polish)**: Depends on all stories

### User Story Dependencies

```
Phase 1 (Setup)
    │
Phase 2 (Foundation: DB + BookmarkButton + API client)
    │
Phase 3 (US1+US2: Toggle + My Bookmarks) ─── MVP
    │
Phase 4 (US3: Update Tracking)
    │
Phase 5 (Polish + E2E)
```

### Parallel Opportunities

- **Phase 2**: T005 (BookmarkButton) + T006 (API client) in parallel
- **Phase 3**: Backend (T008-T021) and frontend (T022-T034) can overlap after DTOs defined
- **Phase 3**: T025 + T026 + T027 (bookmark integration into search/detail/trending) in parallel
- **Phase 5**: All E2E tests (T047-T049) in parallel

---

## Implementation Strategy

### MVP First (Phase 1 + 2 + 3)

1. Phase 1: Shared types
2. Phase 2: DB, BookmarkButton, API client
3. Phase 3: Toggle + My Bookmarks page
4. **STOP and VALIDATE**: Funder can bookmark projects and view them in a list
5. Deploy/demo

### Incremental Delivery

1. **Phase 1+2+3** → Bookmark toggle + list works (MVP)
2. **Phase 4** → Update tracking → Deploy
3. **Phase 5** → E2E + polish → Final release

---

## Notes

- [P] tasks = different files, no dependencies
- US1+US2 grouped — toggle is useless without the list
- Optimistic UI pattern: toggle icon instantly, revert on server error
- Batch check endpoint avoids N+1 queries on search results
- Updates derived from project_audit_log — no separate event system
- Stale bookmarks: LEFT JOIN with SET NULL on delete, show "No longer available"
- Only FUNDER and SUPER_ADMIN can bookmark (RBAC enforced)
