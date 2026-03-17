# Quickstart: Onboarding

**Feature**: 005-onboarding | **Date**: 2026-03-17

## Prerequisites

- Features 001-004 implemented (homepage, registration, project management, bookmarks)

## Setup

```bash
pnpm install
pnpm --filter <backend-app> prisma generate
pnpm --filter <backend-app> prisma migrate dev    # adds user_preferences table
```

## Testing

```bash
pnpm --filter <backend-app> test -- --testPathPattern=preferences
pnpm --filter <web-app> test -- --testPathPattern="onboarding|recommendations"
pnpm --filter <web-app> test:e2e -- --testPathPattern=onboarding
```

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /v1/preferences | Authenticated | Save/update preferences |
| GET | /v1/preferences | Authenticated | Get current preferences |
| GET | /v1/recommendations | Authenticated | Get personalized recommendations |

## Key Modules

**Backend:** `preferences` module (save/get preferences + recommendation generation)
**Frontend:** `onboarding` (post-registration flow), `recommendations` (homepage section), preferences page in account settings
**Shared:** preference + recommendation types in shared-types
