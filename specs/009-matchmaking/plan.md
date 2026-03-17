# Implementation Plan: Matchmaking

**Branch**: `009-matchmaking` | **Date**: 2026-03-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/009-matchmaking/spec.md`

## Summary

Upgrade the basic keyword recommendations (005-onboarding) to an AI-powered matchmaking system. Combines structured field matching (theme overlap, region match, funding gap, sector) with AI vector similarity scoring (Funder preferences vs project embeddings). Produces a composite, deterministic ranking with match scores and reasons. Replaces the existing GET /v1/recommendations endpoint with enhanced scoring. Falls back to structured-only when AI is unavailable.

## Technical Context

**Language/Version**: TypeScript (strict mode)
**Primary Dependencies**: NestJS (backend), React.js + Tailwind CSS (frontend), Prisma ORM, vector store (pgvector from 001-homepage)
**Storage**: PostgreSQL (match_scores cache table optional, preference embeddings)
**Testing**: Jest (unit + integration), Playwright (E2E), React Testing Library
**Target Platform**: Web application (desktop + mobile)
**Project Type**: Web service (monorepo)
**Performance Goals**: Match results within 3 seconds, fallback within 1 second
**Constraints**: Deterministic ordering, structured scores weighted higher than AI, graceful fallback
**Scale/Scope**: Hundreds of projects ranked per Funder, scores cached or computed on-the-fly

## Constitution Check

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Monorepo-First | PASS | Extends existing modules |
| II. Clean Architecture & SOLID | PASS | MatchmakingService in service layer, scoring logic separate from controller |
| III. Modular Architecture | PASS | Backend: `matchmaking` module or extends `preferences` module |
| IV. Strict Type Safety | PASS | Match types in shared-types |
| V. Security by Design | PASS | Auth required, no PII in AI requests |
| VI. Testing Discipline | PASS | Unit + integration + E2E |
| VII. Independent Deployability | PASS | No new apps |
| VIII. Observability-First | PASS | AI scoring logged |
| IX. Shared-Before-Custom | PASS | Reuses vector infrastructure from 001-homepage |
| X. Design Token Management | PASS | Match score display via tokens |

**Gate Result: PASS**

## Project Structure

### Source Code

```text
# Backend
apps/<backend-app>/src/modules/
├── matchmaking/
│   ├── matchmaking.module.ts
│   ├── matchmaking.controller.ts
│   ├── matchmaking.service.ts
│   ├── scoring/
│   │   ├── structured-scorer.ts       # Theme, region, sector, funding gap matching
│   │   ├── ai-scorer.ts              # Vector similarity: preferences vs project embeddings
│   │   └── composite-scorer.ts        # Combines structured + AI scores with weights
│   ├── dto/
│   │   ├── match-result.dto.ts
│   │   └── matches-query.dto.ts
│   ├── __tests__/
│   └── index.ts

# Frontend — enhances existing recommendations
apps/<web-app>/src/features/
├── recommendations/                    # Existing from 005-onboarding, enhanced
│   ├── ui/components/
│   │   ├── RecommendedSection.tsx      # Updated to show match scores + reasons
│   │   ├── MatchScoreBadge.tsx        # New: visual score indicator
│   │   └── MatchReasons.tsx           # New: "Why this was recommended"
│   ├── hooks/
│   │   └── useRecommendations.ts      # Updated to use matchmaking endpoint

# Shared
packages/shared-types/src/
├── matchmaking/
│   ├── match-result.ts
│   ├── match-score.ts
│   └── index.ts

# E2E
tests/e2e/matchmaking/
├── match-ranking.spec.ts
├── match-consistency.spec.ts
└── fallback.spec.ts
```

**Structure Decision**: New `matchmaking` backend module with separate scorer components (structured, AI, composite). Frontend enhances the existing `recommendations` module from 005-onboarding rather than creating a new page. Follows Constitution Principles III, IX.

## Complexity Tracking

No violations.
