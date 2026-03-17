# Tasks: Matchmaking

**Input**: Design documents from `/specs/009-matchmaking/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: TDD — constitution requires 80% coverage.

**Organization**: US1 (Ranked Matches) + US2 (Consistency) grouped as P1 MVP — consistency is a quality requirement of ranking. US3 (Enhanced Recommendations) is P2.

## Format: `[ID] [P?] [Story] Description`

## Path Conventions

- **Backend**: `apps/<backend-app>/src/modules/`
- **Frontend**: `apps/<web-app>/src/features/`
- **Shared types**: `packages/shared-types/src/`

---

## Phase 1: Setup

- [ ] T001 Add match result and match score types in packages/shared-types/src/matchmaking/match-result.ts, match-score.ts, index.ts
- [ ] T002 Re-export from packages/shared-types/src/index.ts

---

## Phase 2: Foundational

- [ ] T003 Create Prisma migration to add preference_embedding (Vector, nullable) to user_preferences table
- [ ] T004 [P] Add matchmaking API client method in packages/api-client/src/matchmaking/ — getMatches(page, pageSize)

---

## Phase 3: US1 + US2 — Ranked Matches + Consistency (Priority: P1) MVP

**Goal**: Funder sees projects ranked by composite score (60% structured + 40% AI). Each project shows match score and reasons. Ranking is deterministic. Fallback to structured-only when AI unavailable.

**Independent Test**: Complete onboarding with "Renewable Energy, East Africa" → view /matches → top results are renewable energy projects in East Africa → revisit → same order.

### Backend — Matchmaking Module

- [ ] T005 [US1] Create matchmaking module scaffold: apps/<backend-app>/src/modules/matchmaking/matchmaking.module.ts, matchmaking.controller.ts, matchmaking.service.ts, scoring/, dto/, index.ts
- [ ] T006 [US1] Create StructuredScorer in apps/<backend-app>/src/modules/matchmaking/scoring/structured-scorer.ts — scores a project against preferences: theme overlap (0.4), region match (0.3), sector match (0.15), funding gap (0.15). Returns 0-1 with match reasons
- [ ] T007 [US1] Create AIScorer in apps/<backend-app>/src/modules/matchmaking/scoring/ai-scorer.ts — computes cosine similarity between preference_embedding and project embedding. Returns 0-1. Returns null if embedding unavailable
- [ ] T008 [US1] Create CompositeScorer in apps/<backend-app>/src/modules/matchmaking/scoring/composite-scorer.ts — combines: 0.6 * structured + 0.4 * AI. If AI null (fallback), uses structured only. Merges match reasons from both scorers
- [ ] T009 [US1] Implement MatchmakingService.getMatches() in apps/<backend-app>/src/modules/matchmaking/matchmaking.service.ts — load user preferences + embedding, score all APPROVED projects, sort by composite DESC then created_at DESC then id ASC (deterministic), paginate, return with scores and reasons
- [ ] T010 [US1] Implement preference embedding generation — on preference save/update (async), embed concatenated preference text, store in preference_embedding column
- [ ] T011 [US2] Implement deterministic tiebreaking in CompositeScorer — composite DESC, created_at DESC, id ASC
- [ ] T012 [US1] Create MatchResultDto and MatchesQueryDto in apps/<backend-app>/src/modules/matchmaking/dto/
- [ ] T013 [US1] Implement GET /v1/matches in MatchmakingController — @Roles(FUNDER, SUPER_ADMIN), paginated, returns scoringMode in meta (composite or structured_only)
- [ ] T014 [US1] Write unit tests for StructuredScorer in apps/<backend-app>/src/modules/matchmaking/__tests__/structured-scorer.spec.ts — theme overlap, region match, sector, funding gap, edge cases
- [ ] T015 [P] [US1] Write unit tests for AIScorer in apps/<backend-app>/src/modules/matchmaking/__tests__/ai-scorer.spec.ts — cosine similarity, null embedding, fallback
- [ ] T016 [P] [US1] Write unit tests for CompositeScorer in apps/<backend-app>/src/modules/matchmaking/__tests__/composite-scorer.spec.ts — weighted combination, AI fallback, reason merging
- [ ] T017 [US2] Write unit tests for deterministic ordering in apps/<backend-app>/src/modules/matchmaking/__tests__/determinism.spec.ts — same input → same output, tiebreaking
- [ ] T018 [US1] Write integration test for GET /v1/matches in apps/<backend-app>/src/modules/matchmaking/__tests__/matchmaking.integration.spec.ts — full scoring pipeline, no preferences (prompt), fallback mode

### Frontend — Enhanced Match Display

- [ ] T019 [US1] Create MatchScoreBadge component in apps/<web-app>/src/features/recommendations/ui/components/MatchScoreBadge.tsx — visual indicator: percentage or color-coded bar (high/medium/low match)
- [ ] T020 [US1] Create MatchReasons component in apps/<web-app>/src/features/recommendations/ui/components/MatchReasons.tsx — list of match reason strings below each project card
- [ ] T021 [US1] Update useRecommendations hook in apps/<web-app>/src/features/recommendations/hooks/useRecommendations.ts — call GET /v1/matches instead of old /v1/recommendations, map response to existing component props
- [ ] T022 [US1] Update RecommendedSection component to display MatchScoreBadge and MatchReasons on each project card
- [ ] T023 [US1] Handle no-preferences state — show "Complete your preferences" prompt (reuse from 005-onboarding)
- [ ] T024 [US1] Handle fallback mode — when meta.scoringMode = "structured_only", show subtle indicator that AI scoring is temporarily unavailable
- [ ] T025 [US1] Write component tests for MatchScoreBadge and MatchReasons in apps/<web-app>/src/features/recommendations/__tests__/MatchDisplay.spec.tsx

**Checkpoint**: US1+US2 complete — ranked matches with scores, reasons, deterministic ordering

---

## Phase 4: US3 — Enhanced Recommendations (Priority: P2)

**Goal**: The homepage "Recommended for you" section uses matchmaking scoring instead of basic keyword matching.

- [ ] T026 [US3] Replace GET /v1/recommendations endpoint logic with a call to MatchmakingService.getMatches() (limit to top 6) — or deprecate the old endpoint and have frontend call /v1/matches?pageSize=6
- [ ] T027 [US3] Verify homepage RecommendedSection now shows AI-scored matches with scores and reasons
- [ ] T028 [US3] Write integration test verifying /v1/recommendations uses matchmaking scoring in apps/<backend-app>/src/modules/matchmaking/__tests__/recommendations-upgrade.spec.ts

**Checkpoint**: US3 complete — homepage recommendations powered by matchmaking

---

## Phase 5: Polish

- [ ] T029 [P] Write E2E test for match ranking in tests/e2e/matchmaking/match-ranking.spec.ts — complete preferences, view matches, verify ordering
- [ ] T030 [P] Write E2E test for match consistency in tests/e2e/matchmaking/match-consistency.spec.ts — view, close, reopen, same order
- [ ] T031 [P] Write E2E test for fallback in tests/e2e/matchmaking/fallback.spec.ts — structured-only mode
- [ ] T032 Run quickstart.md validation

---

## Dependencies & Execution Order

```
Phase 1 (Setup: types)
    │
Phase 2 (Foundation: embedding column + API client)
    │
Phase 3 (US1+US2: Scoring + ranking + display) ─── MVP
    │
Phase 4 (US3: Replace recommendations)
    │
Phase 5 (Polish + E2E)
```

### Parallel Opportunities

- T014+T015+T016 scorer tests in parallel
- T019+T020 match UI components in parallel
- T029+T030+T031 E2E tests in parallel

---

## Implementation Strategy

### MVP First (Phase 1 + 2 + 3)

1. Types + migration + API client
2. 3 scorers + service + endpoint + frontend display
3. **VALIDATE**: Funder sees ranked matches with scores and reasons
4. Deploy

### Incremental

1. **Phase 1+2+3** → Matchmaking works (MVP)
2. **Phase 4** → Replaces basic recommendations → Deploy
3. **Phase 5** → E2E → Final release

---

## Notes

- Composite: 60% structured + 40% AI (configurable weights)
- Structured scoring: theme (0.4) + region (0.3) + sector (0.15) + funding gap (0.15)
- AI scoring: cosine similarity, reuses embedding infrastructure from 001-homepage
- Deterministic: composite DESC → created_at DESC → id ASC
- No cache table — computed on-the-fly. Can add caching later if needed
- Preference embedding generated async on save, stored on user_preferences
