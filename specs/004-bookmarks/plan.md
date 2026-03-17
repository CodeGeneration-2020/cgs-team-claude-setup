# Implementation Plan: Bookmarks

**Branch**: `004-bookmarks` | **Date**: 2026-03-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/004-bookmarks/spec.md`

## Summary

Add bookmark functionality for Funders: toggle bookmarks on project cards and detail pages, view a dedicated "My Bookmarks" page with paginated project cards, and track updates on bookmarked projects by deriving change events from the existing project audit log. The feature extends the existing ProjectCard component with a bookmark toggle, adds a backend bookmarks module, and creates a frontend bookmarks feature module.

## Technical Context

**Language/Version**: TypeScript (strict mode)
**Primary Dependencies**: NestJS (backend), React.js + Tailwind CSS (frontend), Zustand + TanStack Query (state/data), Prisma ORM
**Storage**: PostgreSQL (bookmarks table, bookmark_updates derived from project_audit_log)
**Testing**: Jest (unit + integration), Playwright (E2E), React Testing Library (components)
**Target Platform**: Web application (desktop + tablet + mobile responsive)
**Project Type**: Web service (monorepo)
**Performance Goals**: Bookmark toggle < 1 second perceived, bookmarks page loads < 2 seconds
**Constraints**: Only FUNDER and SUPER_ADMIN can bookmark, optimistic UI for toggle, idempotent operations
**Scale/Scope**: Thousands of bookmarks per user, update tracking within 5 minutes of changes

## Constitution Check

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Monorepo-First | PASS | Adds module to existing apps |
| II. Clean Architecture & SOLID | PASS | Bookmark logic in service, controller delegates |
| III. Modular Architecture | PASS | Backend: `bookmarks` module. Frontend: `bookmarks` feature module |
| IV. Strict Type Safety | PASS | Bookmark types in shared-types |
| V. Security by Design | PASS | @Roles(FUNDER) on bookmark endpoints, auth required |
| VI. Testing Discipline | PASS | Unit + integration + E2E tests |
| VII. Independent Deployability | PASS | No new apps |
| VIII. Observability-First | PASS | Structured logging for bookmark operations |
| IX. Shared-Before-Custom | PASS | Extends existing ProjectCard with bookmark prop |
| X. Design Token Management | PASS | Bookmark icon styling via design tokens |

**Gate Result: PASS**

## Project Structure

### Documentation

```text
specs/004-bookmarks/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── bookmarks.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code

```text
# Backend
apps/<backend-app>/src/modules/
├── bookmarks/
│   ├── bookmarks.module.ts
│   ├── bookmarks.controller.ts
│   ├── bookmarks.service.ts
│   ├── dto/
│   │   ├── bookmark-response.dto.ts
│   │   └── bookmarks-list-query.dto.ts
│   ├── entities/
│   │   └── bookmark.entity.ts
│   ├── __tests__/
│   └── index.ts

# Frontend
apps/<web-app>/src/features/
├── bookmarks/
│   ├── ui/
│   │   ├── pages/
│   │   │   └── MyBookmarksPage.tsx
│   │   └── components/
│   │       ├── BookmarkButton.tsx
│   │       ├── BookmarkList.tsx
│   │       └── BookmarkUpdateBadge.tsx
│   ├── state/
│   │   └── useBookmarkStore.ts
│   ├── hooks/
│   │   ├── useBookmarks.ts
│   │   ├── useBookmarkToggle.ts
│   │   └── useBookmarkUpdates.ts
│   ├── __tests__/
│   └── index.ts

# Shared
packages/shared-types/src/
├── bookmark/
│   ├── bookmark.ts
│   ├── bookmark-update.ts
│   └── index.ts

packages/ui-components/src/
├── BookmarkButton/
│   ├── BookmarkButton.tsx
│   ├── BookmarkButton.stories.tsx
│   └── BookmarkButton.spec.tsx

# E2E
tests/e2e/bookmarks/
├── bookmark-toggle.spec.ts
├── my-bookmarks.spec.ts
└── bookmark-updates.spec.ts
```

**Structure Decision**: Single backend `bookmarks` module handles all bookmark operations. Frontend `bookmarks` feature module for the My Bookmarks page. BookmarkButton is a shared UI component since it appears on ProjectCard across multiple features. Follows Constitution Principles III, IX.

## Complexity Tracking

No violations.
