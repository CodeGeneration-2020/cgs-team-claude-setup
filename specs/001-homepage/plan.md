# Implementation Plan: CFC Homepage

**Branch**: `001-homepage` | **Date**: 2026-03-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-homepage/spec.md`

## Summary

Build the CFC platform homepage as a search-focused landing page with AI-powered semantic search, structured card-based results with filtering and sorting, a curated trending projects section, and a Super Admin interface to manage trending content. The homepage is fully public (no authentication required for visitors). The backend exposes search, filter, sort, and trending management endpoints. The frontend renders a responsive homepage with search input, trending cards, and a results page with filters/sort controls.

## Technical Context

**Language/Version**: TypeScript (strict mode)
**Primary Dependencies**: NestJS (backend), React.js + Tailwind CSS (frontend), Zustand + TanStack Query (state/data), Prisma ORM
**Storage**: PostgreSQL (projects, trending list), vector store or AI service for semantic search embeddings
**Testing**: Jest (unit + integration), Playwright (E2E), React Testing Library (components)
**Target Platform**: Web application (desktop + tablet + mobile responsive)
**Project Type**: Web service (monorepo: backend API + frontend SPA)
**Performance Goals**: Search results within 3 seconds for 95% of queries, homepage loads within 2 seconds
**Constraints**: Public pages must work without authentication, semantic search must handle natural language, trending updates reflect within 30 seconds
**Scale/Scope**: Initial target: hundreds of projects, dozens of concurrent users, scaling to thousands of projects

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Monorepo-First Architecture | PASS | Feature lives in existing monorepo under `/apps/` (web) and backend modules |
| II. Clean Architecture & SOLID | PASS | Search service in service layer, controller delegates only, no business logic in controllers |
| III. Modular Architecture | PASS | Homepage is a frontend feature module (`src/features/homepage/`), backend adds `search` and `trending` domain modules |
| IV. Strict Type Safety | PASS | All DTOs, response shapes, and search params defined in `/packages/shared-types` |
| V. Security by Design | PASS | Homepage endpoints are public (explicitly marked), Super Admin trending management behind Auth Guard + Role Guard |
| VI. Testing Discipline | PASS | Unit tests for search service + trending service, integration tests for endpoints, E2E for homepage flows |
| VII. Independent Deployability | PASS | No new apps created, feature adds modules to existing apps |
| VIII. Observability-First | PASS | Search queries logged with structured logging (Pino), AI usage logged per US-36 requirement |
| IX. Shared-Before-Custom | PASS | Project card component in `/packages/ui-components`, shared types for search/filter DTOs |
| X. Design Token Management | PASS | Tag colors, card styles use design tokens from `/packages/config` theme |

**Gate Result: PASS** — No violations. Proceeding to Phase 0.

## Project Structure

### Documentation (this feature)

```text
specs/001-homepage/
├── spec.md
├── plan.md              # This file
├── research.md          # Phase 0: research decisions
├── data-model.md        # Phase 1: entity schemas
├── quickstart.md        # Phase 1: dev environment setup
├── contracts/           # Phase 1: API contracts
│   ├── search.md
│   └── trending.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit.tasks)
```

### Source Code (repository root)

```text
# Backend (NestJS)
apps/<backend-app>/src/modules/
├── search/
│   ├── search.module.ts
│   ├── search.controller.ts
│   ├── search.service.ts
│   ├── dto/
│   │   ├── search-query.dto.ts
│   │   ├── search-results.dto.ts
│   │   └── search-filters.dto.ts
│   ├── entities/
│   │   └── search-log.entity.ts
│   ├── __tests__/
│   └── index.ts
├── trending/
│   ├── trending.module.ts
│   ├── trending.controller.ts
│   ├── trending.service.ts
│   ├── dto/
│   │   ├── trending-project.dto.ts
│   │   └── update-trending.dto.ts
│   ├── entities/
│   │   └── trending-project.entity.ts
│   ├── __tests__/
│   └── index.ts

# Frontend (React)
apps/<web-app>/src/features/
├── homepage/
│   ├── ui/
│   │   ├── pages/
│   │   │   └── HomePage.tsx
│   │   └── components/
│   │       ├── SearchBar.tsx
│   │       ├── TrendingSection.tsx
│   │       ├── SearchResults.tsx
│   │       ├── ProjectCard.tsx
│   │       ├── FilterPanel.tsx
│   │       ├── SortControls.tsx
│   │       └── ProjectTags.tsx
│   ├── state/
│   │   └── useSearchStore.ts
│   ├── services/
│   │   └── search.service.ts
│   ├── hooks/
│   │   ├── useSearch.ts
│   │   ├── useTrending.ts
│   │   └── useFilters.ts
│   ├── domain/
│   │   └── types.ts
│   ├── __tests__/
│   └── index.ts

# Super Admin trending management
apps/<web-app>/src/features/
├── admin-trending/
│   ├── ui/
│   │   ├── pages/
│   │   │   └── ManageTrendingPage.tsx
│   │   └── components/
│   │       ├── TrendingList.tsx
│   │       ├── AddProjectDialog.tsx
│   │       └── TrendingProjectCard.tsx
│   ├── state/
│   │   └── useTrendingAdminStore.ts
│   ├── services/
│   │   └── trending-admin.service.ts
│   ├── hooks/
│   │   └── useTrendingAdmin.ts
│   ├── __tests__/
│   └── index.ts

# Shared packages
packages/shared-types/src/
├── search/
│   ├── search-query.ts
│   ├── search-result.ts
│   ├── search-filters.ts
│   └── index.ts
├── trending/
│   ├── trending-project.ts
│   └── index.ts
├── project/
│   ├── project-card.ts        # shared card display shape
│   ├── project-status.ts
│   ├── project-tag.ts
│   └── index.ts

packages/ui-components/src/
├── ProjectCard/
│   ├── ProjectCard.tsx
│   ├── ProjectCard.stories.tsx
│   └── ProjectCard.spec.tsx
├── ProjectTag/
│   ├── ProjectTag.tsx
│   ├── ProjectTag.stories.tsx
│   └── ProjectTag.spec.tsx

# Tests
tests/e2e/homepage/
├── search.spec.ts
├── trending.spec.ts
├── filters.spec.ts
└── sort.spec.ts
```

**Structure Decision**: Monorepo with domain-driven modules. Backend adds `search` and `trending` modules under the existing NestJS app. Frontend adds `homepage` and `admin-trending` feature modules. Shared types and UI components live in the existing shared packages. This follows Constitution Principles I, III, and IX.

## Complexity Tracking

No violations — no entries needed.
