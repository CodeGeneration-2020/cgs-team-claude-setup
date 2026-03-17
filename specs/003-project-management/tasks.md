# Tasks: Project Management

**Input**: Design documents from `/specs/003-project-management/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: TDD — constitution requires 80% coverage.

**Organization**: US1 (Create) + US2 (View List) + US3 (Publish) grouped as P1 MVP since create/list/publish form the minimum viable project workflow. US6 (Approve/Reject) is a separate P1 phase since it requires Super Admin. US4 (Edit), US5 (Delete), US7 (Tags) are P2.

## Format: `[ID] [P?] [Story] Description`

## Path Conventions

- **Backend**: `apps/<backend-app>/src/modules/`
- **Frontend**: `apps/<web-app>/src/features/`
- **Shared types**: `packages/shared-types/src/`
- **API client**: `packages/api-client/src/`
- **E2E tests**: `tests/e2e/project-management/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Shared types, taxonomy types, project types, API client

- [ ] T001 Add project create/update/response types in packages/shared-types/src/project/create-project.ts, update-project.ts, project-response.ts
- [ ] T002 [P] Extend ProjectStatus enum with full lifecycle values (DRAFT, PENDING_APPROVAL, APPROVED, REJECTED, ARCHIVED) in packages/shared-types/src/project/project-status.ts
- [ ] T003 [P] Add TaxonomyCategory enum and TaxonomyValue type in packages/shared-types/src/taxonomy/taxonomy-category.ts, taxonomy-value.ts, index.ts
- [ ] T004 [P] Add ProjectAction enum in packages/shared-types/src/project/project-action.ts
- [ ] T005 [P] Add admin review types (ReviewDecision) in packages/shared-types/src/admin/review-decision.ts, index.ts
- [ ] T006 Re-export all new types from packages/shared-types/src/index.ts barrel

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Database schema, taxonomy seed, API client — MUST complete before user stories

- [ ] T007 Create Prisma migration to extend projects table: add description (Text), owner_id (FK to users), thematic_areas (String[]), stakeholder_sector, implementer_type, lifecycle_status, region, is_local_ngo (Boolean), timeline, partners, sustainability_info, rejection_reason (nullable), reviewed_by (FK nullable), reviewed_at (nullable), published_at (nullable), version (Int default 1). Add indexes on owner_id, status, created_at
- [ ] T008 [P] Create Prisma migration for taxonomy_values table: id, category (TaxonomyCategory enum), value (String), display_order (Int), is_active (Boolean default true), created_at. Add unique index on (category, value), index on category
- [ ] T009 [P] Create Prisma migration for project_audit_log table: id, project_id (nullable FK), user_id (FK), action (ProjectAction enum), changed_fields (JSON nullable), metadata (JSON nullable), created_at. Add indexes on project_id, user_id, created_at
- [ ] T010 Create taxonomy seed script in prisma/seed-taxonomy.ts — seed all taxonomy values from client-provided data for categories: THEMATIC_AREA, STAKEHOLDER_SECTOR, IMPLEMENTER_TYPE, REGION, LIFECYCLE_STATUS
- [ ] T011 Create taxonomy module scaffold: apps/<backend-app>/src/modules/taxonomy/taxonomy.module.ts, taxonomy.controller.ts, taxonomy.service.ts, entities/taxonomy-value.entity.ts, index.ts
- [ ] T012 Implement TaxonomyService.getAllGrouped() in apps/<backend-app>/src/modules/taxonomy/taxonomy.service.ts — return values grouped by category, ordered by display_order, filter is_active=true
- [ ] T013 Implement GET /v1/taxonomy in TaxonomyController — @Public() endpoint, returns grouped taxonomy values
- [ ] T014 [P] Write unit tests for TaxonomyService in apps/<backend-app>/src/modules/taxonomy/__tests__/taxonomy.service.spec.ts
- [ ] T015 Add project management API client methods in packages/api-client/src/projects/ — createProject(), getMyProjects(), getProject(), updateProject(), publishProject(), deleteProject(), getTaxonomy()
- [ ] T016 [P] Add admin review API client methods in packages/api-client/src/admin/ — getPendingProjects(), approveProject(), rejectProject()

**Checkpoint**: Foundation ready — DB schema, taxonomy seeded, API client stubs

---

## Phase 3: US1 + US2 + US3 — Create + View List + Publish (Priority: P1) MVP

**Goal**: Org Admin can create a project via structured form with taxonomy fields, see it in "My Projects" list, and publish it for approval (status → Pending Approval). Audit log entries generated.

**Independent Test**: Log in as Org Admin → create project → see it in My Projects → publish → verify status is Pending Approval.

### Backend — Projects Module

- [ ] T017 [US1] Create projects module scaffold: apps/<backend-app>/src/modules/projects/projects.module.ts, projects.controller.ts, projects.service.ts, dto/, entities/, index.ts
- [ ] T018 [P] [US1] Create Project entity in apps/<backend-app>/src/modules/projects/entities/project.entity.ts — maps to extended projects table
- [ ] T019 [P] [US1] Create ProjectAuditLog entity in apps/<backend-app>/src/modules/projects/entities/project-audit-log.entity.ts
- [ ] T020 [US1] Create CreateProjectDto in apps/<backend-app>/src/modules/projects/dto/create-project.dto.ts — validates all required fields, title max 200, description max 5000, thematicAreas min 1 item, taxonomy fields validated against TaxonomyService
- [ ] T021 [P] [US2] Create ProjectListQueryDto in apps/<backend-app>/src/modules/projects/dto/project-list-query.dto.ts — validates status filter, page, pageSize
- [ ] T022 [P] [US1] Create ProjectResponseDto in apps/<backend-app>/src/modules/projects/dto/project-response.dto.ts — response shape per contracts/projects.md
- [ ] T023 [US1] Implement ProjectsService.create() in apps/<backend-app>/src/modules/projects/projects.service.ts — validate taxonomy values exist, save project with owner_id from current user, set status PENDING_APPROVAL, write CREATED audit log entry
- [ ] T024 [US2] Implement ProjectsService.getMyProjects() — query projects by owner_id with status filter, pagination, return list with status indicators
- [ ] T025 [US3] Implement ProjectsService.publish() — validate project belongs to current user, change status to PENDING_APPROVAL, set published_at, write PUBLISHED audit log entry
- [ ] T026 [US1] Implement POST /v1/projects in ProjectsController — @Roles(ORG_ADMIN), delegates to service, returns 201
- [ ] T027 [US2] Implement GET /v1/projects/my in ProjectsController — @Roles(ORG_ADMIN), delegates to service with pagination
- [ ] T028 [US3] Implement POST /v1/projects/:id/publish in ProjectsController — @Roles(ORG_ADMIN), validates ownership, delegates to service
- [ ] T029 Implement GET /v1/projects/:id in ProjectsController — public for APPROVED projects, owner/admin for other statuses
- [ ] T030 [US1] Write unit tests for ProjectsService.create() in apps/<backend-app>/src/modules/projects/__tests__/projects.service.spec.ts — valid creation, missing fields, invalid taxonomy values, audit log generation
- [ ] T031 [P] [US2] Write unit tests for ProjectsService.getMyProjects() — pagination, status filtering, ownership
- [ ] T032 [P] [US3] Write unit tests for ProjectsService.publish() — valid publish, not owner (blocked), audit log
- [ ] T033 [US1] Write integration tests for project CRUD endpoints in apps/<backend-app>/src/modules/projects/__tests__/projects.integration.spec.ts

### Frontend — Project Management Module

- [ ] T034 [US1] Create project-management feature module scaffold: apps/<web-app>/src/features/project-management/ with ui/pages/, ui/components/, state/, services/, hooks/, index.ts
- [ ] T035 [US1] Create useTaxonomy hook in apps/<web-app>/src/features/project-management/hooks/useTaxonomy.ts — fetches taxonomy values via TanStack Query, caches response
- [ ] T036 [US1] Create TaxonomySelect component in apps/<web-app>/src/features/project-management/ui/components/TaxonomySelect.tsx — dropdown populated from taxonomy values, supports single and multi-select
- [ ] T037 [US1] Create ProjectForm component in apps/<web-app>/src/features/project-management/ui/components/ProjectForm.tsx — structured form with all fields, taxonomy selects, client-side validation matching backend rules, supports create and edit modes
- [ ] T038 [US1] Create CreateProjectPage in apps/<web-app>/src/features/project-management/ui/pages/CreateProjectPage.tsx — renders ProjectForm in create mode, handles submit via API client, redirects to My Projects on success
- [ ] T039 [US3] Create PublishProjectDialog in apps/<web-app>/src/features/project-management/ui/components/PublishProjectDialog.tsx — confirmation modal "Publish this project?"
- [ ] T040 [US2] Create ProjectStatusBadge component in apps/<web-app>/src/features/project-management/ui/components/ProjectStatusBadge.tsx — color-coded badge for Draft, Pending Approval, Approved, Rejected, Archived
- [ ] T041 [US2] Create ProjectList component in apps/<web-app>/src/features/project-management/ui/components/ProjectList.tsx — renders list of user's projects with title, thematic area, location, funding, status badge, action buttons
- [ ] T042 [US2] Create useMyProjects hook in apps/<web-app>/src/features/project-management/hooks/useMyProjects.ts — fetches user's projects via TanStack Query with status filter and pagination
- [ ] T043 [US2] Create MyProjectsPage in apps/<web-app>/src/features/project-management/ui/pages/MyProjectsPage.tsx — renders ProjectList, status filter tabs, pagination. Route: /my-projects (ORG_ADMIN only)
- [ ] T044 [US1] Write component tests for ProjectForm in apps/<web-app>/src/features/project-management/__tests__/ProjectForm.spec.tsx — validation, taxonomy select, submit
- [ ] T045 [P] [US2] Write component tests for ProjectList and MyProjectsPage in apps/<web-app>/src/features/project-management/__tests__/MyProjects.spec.tsx
- [ ] T046 Add "My Projects" and "Create Project" links to Org Admin navigation

**Checkpoint**: US1+US2+US3 complete — Org Admin can create, view, and publish projects

---

## Phase 4: US6 — Approve or Reject a Project (Priority: P1)

**Goal**: Super Admin reviews pending projects in a queue and can approve (→ public) or reject (→ with reason). Concurrent decisions handled with optimistic locking.

**Independent Test**: Log in as Super Admin → open review queue → approve one project → reject another with reason → verify statuses updated.

### Backend — Admin Review Module

- [ ] T047 [US6] Create admin-review module scaffold: apps/<backend-app>/src/modules/admin-review/admin-review.module.ts, admin-review.controller.ts, admin-review.service.ts, dto/, index.ts
- [ ] T048 [US6] Create ReviewDecisionDto in apps/<backend-app>/src/modules/admin-review/dto/review-decision.dto.ts — reason required for rejection (min 10, max 2000 chars)
- [ ] T049 [P] [US6] Create PendingProjectsQueryDto in apps/<backend-app>/src/modules/admin-review/dto/pending-projects-query.dto.ts — page, pageSize, sortBy, sortOrder
- [ ] T050 [US6] Implement AdminReviewService.getPending() in apps/<backend-app>/src/modules/admin-review/admin-review.service.ts — query projects with PENDING_APPROVAL status, join owner info, paginate
- [ ] T051 [US6] Implement AdminReviewService.approve() — check project is PENDING_APPROVAL (optimistic lock), change status to APPROVED, set reviewed_by/reviewed_at, write APPROVED audit log. Return 409 if already reviewed
- [ ] T052 [US6] Implement AdminReviewService.reject() — check project is PENDING_APPROVAL (optimistic lock), change status to REJECTED, set rejection_reason + reviewed_by/reviewed_at, write REJECTED audit log. Return 409 if already reviewed
- [ ] T053 [US6] Implement GET /v1/admin/projects/pending in AdminReviewController — @Roles(SUPER_ADMIN)
- [ ] T054 [US6] Implement POST /v1/admin/projects/:id/approve in AdminReviewController — @Roles(SUPER_ADMIN)
- [ ] T055 [US6] Implement POST /v1/admin/projects/:id/reject in AdminReviewController — @Roles(SUPER_ADMIN)
- [ ] T056 [US6] Write unit tests for AdminReviewService in apps/<backend-app>/src/modules/admin-review/__tests__/admin-review.service.spec.ts — approve, reject, concurrent decision (409), missing project (404)
- [ ] T057 [P] [US6] Write integration tests for admin review endpoints in apps/<backend-app>/src/modules/admin-review/__tests__/admin-review.integration.spec.ts

### Frontend — Admin Review Module

- [ ] T058 [US6] Create admin-review feature module scaffold: apps/<web-app>/src/features/admin-review/ with ui/pages/, ui/components/, hooks/, index.ts
- [ ] T059 [US6] Create useAdminReview hook in apps/<web-app>/src/features/admin-review/hooks/useAdminReview.ts — fetches pending projects, approve/reject mutations
- [ ] T060 [US6] Create ProjectReviewCard component in apps/<web-app>/src/features/admin-review/ui/components/ProjectReviewCard.tsx — shows project details with Approve and Reject buttons
- [ ] T061 [US6] Create RejectReasonDialog in apps/<web-app>/src/features/admin-review/ui/components/RejectReasonDialog.tsx — textarea for rejection reason (min 10 chars), confirm/cancel
- [ ] T062 [US6] Create ReviewQueue component in apps/<web-app>/src/features/admin-review/ui/components/ReviewQueue.tsx — lists pending projects with pagination and sort
- [ ] T063 [US6] Create ReviewQueuePage in apps/<web-app>/src/features/admin-review/ui/pages/ReviewQueuePage.tsx — route: /admin/review (SUPER_ADMIN only)
- [ ] T064 [US6] Write component tests for ReviewQueue and RejectReasonDialog in apps/<web-app>/src/features/admin-review/__tests__/
- [ ] T065 Add "Review Queue" link to Super Admin navigation with pending count badge

**Checkpoint**: US6 complete — Super Admin can approve/reject projects, approved projects visible in search

---

## Phase 5: US4 — Edit a Project (Priority: P2)

**Goal**: Org Admin can edit owned projects. Edits to approved projects flagged for re-review. Field-level audit log captured.

**Independent Test**: Open My Projects → select project → edit fields → save → verify changes persisted and audit log entry created.

- [ ] T066 [US4] Create UpdateProjectDto in apps/<backend-app>/src/modules/projects/dto/update-project.dto.ts — all fields optional, same validation rules as create
- [ ] T067 [US4] Implement ProjectsService.update() in apps/<backend-app>/src/modules/projects/projects.service.ts — validate ownership, compute field-level diff, save changes, write EDITED audit log with changed_fields JSON. If approved project and critical fields changed, add a flag/note
- [ ] T068 [US4] Implement PATCH /v1/projects/:id in ProjectsController — @Roles(ORG_ADMIN), validates ownership
- [ ] T069 [US4] Implement re-publish flow for rejected projects — after editing a REJECTED project, Org Admin can publish again (status → PENDING_APPROVAL, write REPUBLISHED audit log)
- [ ] T070 [US4] Create EditProjectPage in apps/<web-app>/src/features/project-management/ui/pages/EditProjectPage.tsx — renders ProjectForm in edit mode with pre-filled values, route: /my-projects/:id/edit
- [ ] T071 [US4] Create useProjectForm hook in apps/<web-app>/src/features/project-management/hooks/useProjectForm.ts — loads project data, manages form state, submit update
- [ ] T072 [US4] Display rejection reason banner on rejected projects in My Projects — show reason + "Edit and Resubmit" button
- [ ] T073 [US4] Write unit tests for ProjectsService.update() — valid edit, not owner (blocked), field diff, re-publish rejected
- [ ] T074 [P] [US4] Write component tests for EditProjectPage in apps/<web-app>/src/features/project-management/__tests__/EditProject.spec.tsx

**Checkpoint**: US4 complete — projects editable, diffs logged, rejected projects can be resubmitted

---

## Phase 6: US5 — Delete a Project (Priority: P2)

**Goal**: Org Admin can delete owned projects. Hard delete with audit snapshot. Confirmation required.

**Independent Test**: Open My Projects → select project → delete → confirm → verify removed from list and search.

- [ ] T075 [US5] Implement ProjectsService.delete() in apps/<backend-app>/src/modules/projects/projects.service.ts — validate ownership, capture project snapshot in audit metadata, hard delete record, write DELETED audit log
- [ ] T076 [US5] Implement DELETE /v1/projects/:id in ProjectsController — @Roles(ORG_ADMIN), validates ownership
- [ ] T077 [US5] Create DeleteProjectDialog in apps/<web-app>/src/features/project-management/ui/components/DeleteProjectDialog.tsx — confirmation modal "Are you sure you want to delete this project?"
- [ ] T078 [US5] Integrate delete action into ProjectList and project detail page — show delete button, trigger dialog, remove from list on success
- [ ] T079 [US5] Write unit tests for ProjectsService.delete() — valid delete, not owner (blocked), audit snapshot
- [ ] T080 [P] [US5] Write component test for DeleteProjectDialog in apps/<web-app>/src/features/project-management/__tests__/DeleteProject.spec.tsx

**Checkpoint**: US5 complete — projects deletable with confirmation and audit trail

---

## Phase 7: US7 — Predefined Taxonomy Tags (Priority: P2)

**Goal**: Tags from predefined taxonomy appear on project cards in search results and on project detail pages. Tags power the search filter options.

**Independent Test**: Create project with taxonomy tags → verify tags appear on card in search results and on detail page → verify filter options include those taxonomy values.

- [ ] T081 [US7] Ensure ProjectCard (from packages/ui-components) renders taxonomy tags — verify thematic area and status tags from project data display correctly
- [ ] T082 [US7] Ensure project detail page displays taxonomy tags — same tags as on cards
- [ ] T083 [US7] Verify search filters (from 001-homepage) populate from GET /v1/taxonomy and correctly filter projects by taxonomy values
- [ ] T084 [US7] Add validation in ProjectsService.create() and .update() that rejects taxonomy values not in the taxonomy_values table — return clear error: "'XYZ' is not a valid thematic area"
- [ ] T085 [US7] Write integration test verifying taxonomy validation rejects invalid values in apps/<backend-app>/src/modules/projects/__tests__/taxonomy-validation.spec.ts

**Checkpoint**: US7 complete — tags display on cards and detail pages, filters work with taxonomy data

---

## Phase 8: Polish & Cross-Cutting Concerns

- [ ] T086 [P] Add responsive layout for project form, My Projects page, and review queue — mobile 375px, tablet 768px, desktop 1440px
- [ ] T087 [P] Add loading states and skeleton screens for My Projects list and review queue during data fetching
- [ ] T088 [P] Write E2E test for project creation flow in tests/e2e/project-management/create-project.spec.ts
- [ ] T089 [P] Write E2E test for edit project flow in tests/e2e/project-management/edit-project.spec.ts
- [ ] T090 [P] Write E2E test for delete project flow in tests/e2e/project-management/delete-project.spec.ts
- [ ] T091 [P] Write E2E test for My Projects page in tests/e2e/project-management/my-projects.spec.ts
- [ ] T092 [P] Write E2E test for admin review flow in tests/e2e/project-management/admin-review.spec.ts
- [ ] T093 [P] Write E2E test for taxonomy validation in tests/e2e/project-management/taxonomy.spec.ts
- [ ] T094 Run quickstart.md validation — verify all setup, seed, and test commands work

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies — start immediately
- **Phase 2 (Foundation)**: Depends on Phase 1 — BLOCKS all user stories
- **Phase 3 (US1+US2+US3)**: Depends on Phase 2 — MVP project creation/list/publish
- **Phase 4 (US6)**: Depends on Phase 3 (projects must exist to be reviewed). Can start backend in parallel with Phase 3 frontend
- **Phase 5 (US4)**: Depends on Phase 3 (extends project service with update). Can run in parallel with Phase 4
- **Phase 6 (US5)**: Depends on Phase 3 (extends project service with delete). Can run in parallel with Phases 4-5
- **Phase 7 (US7)**: Depends on Phase 2 (taxonomy) + Phase 3 (project creation). Mostly verification
- **Phase 8 (Polish)**: Depends on all desired user stories

### User Story Dependencies

```
Phase 1 (Setup)
    │
Phase 2 (Foundation: DB + taxonomy + API client)
    │
Phase 3 (US1+US2+US3: Create + List + Publish) ─── MVP
    │
    ├──────────┬──────────┬──────────┐
    │          │          │          │
Phase 4     Phase 5    Phase 6    Phase 7
US6 Review  US4 Edit   US5 Delete US7 Tags
(P1)        (P2)       (P2)       (P2)
    │          │          │          │
    └──────────┴──────────┴──────────┘
                    │
                Phase 8 (Polish + E2E)
```

### Parallel Opportunities

- **Phase 1**: T001-T006 all type definitions in parallel
- **Phase 2**: T007-T009 migrations in parallel, T015-T016 API client in parallel
- **Phase 3**: DTOs (T020-T022) in parallel, service tests (T030-T032) in parallel
- **Phases 4+5+6+7**: All can run in parallel after Phase 3
- **Phase 8**: All E2E tests (T088-T093) in parallel

---

## Parallel Example: Phase 3

```bash
# After T017 (module scaffold), launch in parallel:
Task T018: "Create Project entity"
Task T019: "Create ProjectAuditLog entity"

# After entities, DTOs in parallel:
Task T020: "Create CreateProjectDto"
Task T021: "Create ProjectListQueryDto"
Task T022: "Create ProjectResponseDto"

# Service tests in parallel:
Task T030: "Unit tests for create"
Task T031: "Unit tests for getMyProjects"
Task T032: "Unit tests for publish"
```

---

## Implementation Strategy

### MVP First (Phase 1 + 2 + 3 = US1+US2+US3)

1. Phase 1: Shared types
2. Phase 2: DB schema, taxonomy seed, API client
3. Phase 3: Create + list + publish projects
4. **STOP and VALIDATE**: Org Admin can create a project, see it in My Projects, publish for review
5. Deploy/demo MVP

### Incremental Delivery

1. **Phase 1+2+3** → Project creation works (MVP)
2. **Phase 4** → Super Admin review queue → Deploy
3. **Phase 5** → Edit projects → Deploy
4. **Phase 6** → Delete projects → Deploy
5. **Phase 7** → Taxonomy tags verified → Deploy
6. **Phase 8** → E2E tests, polish → Final release

### Parallel Team Strategy

With 2 developers after Phase 2:
- **Dev A**: Phase 3 (create/list/publish) → Phase 5 (edit) → Phase 6 (delete)
- **Dev B**: Phase 4 (admin review) → Phase 7 (tags verification)
- **Together**: Phase 8 (E2E + polish)

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps to spec.md user stories
- US1+US2+US3 grouped as MVP because create/list/publish are inseparable for a working project workflow
- Taxonomy validation in service layer — ProjectsService calls TaxonomyService to verify values exist
- Audit log writes are fire-and-forget to avoid blocking user-facing responses
- Hard delete for US5 with audit snapshot — no soft delete
- Optimistic locking in US6 for concurrent Super Admin decisions (409 Conflict)
- Rejection reason stored on project entity and visible in My Projects for Org Admin
