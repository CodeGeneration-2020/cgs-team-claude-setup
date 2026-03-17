# Quickstart: Matchmaking

**Feature**: 009-matchmaking | **Date**: 2026-03-17

## Prerequisites

- Features 001 (search with embeddings) and 005 (onboarding with preferences) implemented
- Same AI embedding model configured as 001-homepage

## Setup

```bash
pnpm install
pnpm --filter <backend-app> prisma generate
pnpm --filter <backend-app> prisma migrate dev    # adds preference_embedding to user_preferences
```

## Testing

```bash
pnpm --filter <backend-app> test -- --testPathPattern=matchmaking
pnpm --filter <web-app> test -- --testPathPattern=recommendations
pnpm --filter <web-app> test:e2e -- --testPathPattern=matchmaking
```

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | /v1/matches | FUNDER | Ranked project matches (replaces /v1/recommendations) |

## Key Modules

**Backend:** `matchmaking` module with 3 scorers (structured, AI, composite)
**Frontend:** Enhanced `recommendations` module with match scores + reasons
