# Implementation Plan: Project Management

**Branch**: `003-project-management` | **Date**: 2026-03-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/003-project-management/spec.md`

## Summary

Implement full project lifecycle management for the CFC platform: Org Admins create projects via structured forms with predefined taxonomy fields, publish them for review, edit and delete owned projects. Super Admins approve or reject submissions from a review queue. All lifecycle actions are audit-logged. Predefined taxonomy values are seeded from a client-provided list and enforced on all taxonomy fields. The frontend provides Org Admin project CRUD pages and a Super Admin review queue.

## Technical Context

**Language/Version**: TypeScript (strict mode)
**Primary Dependencies**: NestJS (backend), React.js + Tailwind CSS (frontend), Zustand + TanStack Query (state/data), Prisma ORM
**Storage**: PostgreSQL (projects, taxonomy, audit logs)
**Testing**: Jest (unit + integration), Playwright (E2E), React Testing Library (components)
**Target Platform**: Web application (desktop + tablet + mobile responsive)
**Project Type**: Web service (monorepo: backend API + frontend SPA)
**Performance Goals**: My Projects page loads within 2 seconds, project form submission within 3 seconds
**Constraints**: Only Org Admin can CRUD projects (RBAC from 002-registration-roles), only Super Admin can approve/reject, taxonomy values are immutable via UI
**Scale/Scope**: Hundreds to thousands of projects, dozens of concurrent Org Admins

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Monorepo-First | PASS | Adds modules to existing backend + frontend apps |
| II. Clean Architecture & SOLID | PASS | Business logic in ProjectService, controllers delegate only, validation in DTOs via Pipes |
| III. Modular Architecture | PASS | Backend: `projects` module (domain-driven). Frontend: `project-management` + `admin-review` feature modules |
| IV. Strict Type Safety | PASS | All DTOs and project types in `/packages/shared-types` |
| V. Security by Design | PASS | Project CRUD behind @Roles(ORG_ADMIN), approval behind @Roles(SUPER_ADMIN), all input validated via class-validator |
| VI. Testing Discipline | PASS | Unit tests for services, integration for endpoints, E2E for flows |
| VII. Independent Deployability | PASS | No new apps |
| VIII. Observability-First | PASS | Audit log for all lifecycle events, structured Pino logging |
| IX. Shared-Before-Custom | PASS | Project types in shared-types, ProjectCard reused from 001-homepage |
| X. Design Token Management | PASS | Status badges, form styling via design tokens |

**Gate Result: PASS** — No violations.

## Project Structure

### Documentation (this feature)

```text
specs/003-project-management/
├── spec.md
├── plan.md              # This file
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── projects.md
│   └── admin-review.md
├── checklists/
│   └── requirements.md
└── tasks.md             # /speckit.tasks output
```

### Source Code (repository root)

```text
# Backend (NestJS)
apps/<backend-app>/src/modules/
├── projects/
│   ├── projects.module.ts
│   ├── projects.controller.ts
│   ├── projects.service.ts
│   ├── dto/
│   │   ├── create-project.dto.ts
│   │   ├── update-project.dto.ts
│   │   ├── project-response.dto.ts
│   │   └── project-list-query.dto.ts
│   ├── entities/
│   │   ├── project.entity.ts
│   │   └── project-audit-log.entity.ts
│   ├── __tests__/
│   └── index.ts
├── taxonomy/
│   ├── taxonomy.module.ts
│   ├── taxonomy.controller.ts
│   ├── taxonomy.service.ts
│   ├── entities/
│   │   └── taxonomy-value.entity.ts
│   ├── __tests__/
│   └── index.ts
├── admin-review/
│   ├── admin-review.module.ts
│   ├── admin-review.controller.ts
│   ├── admin-review.service.ts
│   ├── dto/
│   │   ├── review-decision.dto.ts
│   │   └── pending-projects-query.dto.ts
│   ├── __tests__/
│   └── index.ts

# Frontend (React)
apps/<web-app>/src/features/
├── project-management/
│   ├── ui/
│   │   ├── pages/
│   │   │   ├── MyProjectsPage.tsx
│   │   │   ├── CreateProjectPage.tsx
│   │   │   ├── EditProjectPage.tsx
│   │   │   └── ProjectDetailPage.tsx
│   │   └── components/
│   │       ├── ProjectForm.tsx
│   │       ├── ProjectList.tsx
│   │       ├── ProjectStatusBadge.tsx
│   │       ├── TaxonomySelect.tsx
│   │       ├── DeleteProjectDialog.tsx
│   │       └── PublishProjectDialog.tsx
│   ├── state/
│   │   └── useProjectStore.ts
│   ├── services/
│   │   └── project.service.ts
│   ├── hooks/
│   │   ├── useMyProjects.ts
│   │   ├── useProjectForm.ts
│   │   └── useTaxonomy.ts
│   ├── __tests__/
│   └── index.ts
├── admin-review/
│   ├── ui/
│   │   ├── pages/
│   │   │   └── ReviewQueuePage.tsx
│   │   └── components/
│   │       ├── ReviewQueue.tsx
│   │       ├── ProjectReviewCard.tsx
│   │       └── RejectReasonDialog.tsx
│   ├── hooks/
│   │   └── useAdminReview.ts
│   ├── __tests__/
│   └── index.ts

# Shared packages
packages/shared-types/src/
├── project/
│   ├── create-project.ts
│   ├── update-project.ts
│   ├── project-response.ts
│   ├── project-status.ts          # extended with full lifecycle
│   └── index.ts
├── taxonomy/
│   ├── taxonomy-category.ts
│   ├── taxonomy-value.ts
│   └── index.ts
├── admin/
│   ├── review-decision.ts
│   └── index.ts

# Seed data
prisma/
├── seed-taxonomy.ts               # Taxonomy seed script

# E2E tests
tests/e2e/project-management/
├── create-project.spec.ts
├── edit-project.spec.ts
├── delete-project.spec.ts
├── my-projects.spec.ts
├── admin-review.spec.ts
└── taxonomy.spec.ts
```

**Structure Decision**: Three backend modules: `projects` (Org Admin CRUD), `taxonomy` (predefined values), `admin-review` (Super Admin queue). Two frontend feature modules: `project-management` (Org Admin) and `admin-review` (Super Admin). Shared project types extended in shared-types. Follows Constitution Principles I, III, IX.

## Complexity Tracking

No violations — no entries needed.
