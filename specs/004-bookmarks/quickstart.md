# Quickstart: Bookmarks

**Feature**: 004-bookmarks | **Date**: 2026-03-17

## Prerequisites

- Features 001-003 implemented (homepage, registration, project management)
- Funder accounts exist for testing

## Setup

```bash
pnpm install
pnpm --filter <backend-app> prisma generate
pnpm --filter <backend-app> prisma migrate dev    # adds bookmarks table
```

## Testing

```bash
pnpm --filter <backend-app> test -- --testPathPattern=bookmarks
pnpm --filter <web-app> test -- --testPathPattern=bookmarks
pnpm --filter <web-app> test:e2e -- --testPathPattern=bookmarks
```

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /v1/bookmarks/:projectId | FUNDER | Bookmark a project |
| DELETE | /v1/bookmarks/:projectId | FUNDER | Remove bookmark |
| GET | /v1/bookmarks | FUNDER | List my bookmarks |
| GET | /v1/bookmarks/updates | FUNDER | Updates on bookmarked projects |
| POST | /v1/bookmarks/:projectId/mark-seen | FUNDER | Mark updates as seen |
| GET | /v1/bookmarks/check?projectIds=... | FUNDER | Batch check bookmark state |

## Key Modules

**Backend:** `bookmarks` module (toggle, list, updates, batch check)
**Frontend:** `bookmarks` feature module (My Bookmarks page, BookmarkButton)
**Shared:** BookmarkButton in ui-components, bookmark types in shared-types
