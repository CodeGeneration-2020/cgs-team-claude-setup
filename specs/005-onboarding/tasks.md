# Tasks: Onboarding

**Input**: Design documents from `/specs/005-onboarding/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: TDD — constitution requires 80% coverage.

**Organization**: US1 (Complete Onboarding) + US2 (Skip Onboarding) grouped as P1 MVP — they're two sides of the same flow. US3 (Update Preferences) and US4 (Recommendations) are P2.

## Format: `[ID] [P?] [Story] Description`

## Path Conventions

- **Backend**: `apps/<backend-app>/src/modules/`
- **Frontend**: `apps/<web-app>/src/features/`
- **Shared types**: `packages/shared-types/src/`
- **API client**: `packages/api-client/src/`
- **E2E tests**: `tests/e2e/onboarding/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Shared types and API client

- [ ] T001 Add user preferences types in packages/shared-types/src/preferences/user-preferences.ts — SavePreferencesRequest (areasOfInterest, geographicPreferences, fundingFocus), PreferencesResponse
- [ ] T002 [P] Add recommendation type in packages/shared-types/src/preferences/recommendation.ts — projectId, title, organizationName, thematicAreas, location, shortDescription, matchScore, matchReasons
- [ ] T003 Re-export from packages/shared-types/src/preferences/index.ts and packages/shared-types/src/index.ts

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Database schema and API client

- [ ] T004 Create Prisma migration for user_preferences table: id (CUID), user_id (FK to users, unique), areas_of_interest (Text nullable), geographic_preferences (Text nullable), funding_focus (Text nullable), is_onboarding_completed (Boolean default false), completed_at (DateTime nullable), created_at, updated_at. Add unique index on user_id
- [ ] T005 [P] Add preferences API client methods in packages/api-client/src/preferences/ — savePreferences(), getPreferences(), getRecommendations()

**Checkpoint**: Foundation ready

---

## Phase 3: US1 + US2 — Complete / Skip Onboarding (Priority: P1) MVP

**Goal**: After registration + role selection, user is shown onboarding questionnaire with 3 free-text questions. User can answer and submit (preferences saved) or skip (redirected to platform without saving). Platform works fully regardless.

**Independent Test**: Register → select role → see onboarding → answer questions → submit → verify saved. Register → select role → skip → verify redirect to platform with all features working.

### Backend — Preferences Module

- [ ] T006 [US1] Create preferences module scaffold: apps/<backend-app>/src/modules/preferences/preferences.module.ts, preferences.controller.ts, preferences.service.ts, dto/, entities/, index.ts
- [ ] T007 [US1] Create UserPreferences entity in apps/<backend-app>/src/modules/preferences/entities/user-preferences.entity.ts — maps to user_preferences table
- [ ] T008 [P] [US1] Create SavePreferencesDto in apps/<backend-app>/src/modules/preferences/dto/save-preferences.dto.ts — validates areasOfInterest (max 2000), geographicPreferences (max 2000), fundingFocus (max 2000), at least one field non-empty
- [ ] T009 [P] [US1] Create PreferencesResponseDto in apps/<backend-app>/src/modules/preferences/dto/preferences-response.dto.ts
- [ ] T010 [US1] Implement PreferencesService.savePreferences() in apps/<backend-app>/src/modules/preferences/preferences.service.ts — upsert user_preferences record, set is_onboarding_completed=true, set completed_at
- [ ] T011 [US1] Implement PreferencesService.getPreferences() — return user's preferences or null if none exist
- [ ] T012 [US1] Implement POST /v1/preferences in PreferencesController — requires auth, delegates to service
- [ ] T013 [US1] Implement GET /v1/preferences in PreferencesController — requires auth, returns preferences or null
- [ ] T014 [US1] Write unit tests for PreferencesService in apps/<backend-app>/src/modules/preferences/__tests__/preferences.service.spec.ts — save, upsert, get, null case
- [ ] T015 [P] [US1] Write integration tests for preferences endpoints in apps/<backend-app>/src/modules/preferences/__tests__/preferences.integration.spec.ts

### Frontend — Onboarding Flow

- [ ] T016 [US1] Create onboarding feature module scaffold: apps/<web-app>/src/features/onboarding/ with ui/pages/, ui/components/, hooks/, index.ts
- [ ] T017 [US1] Create OnboardingQuestion component in apps/<web-app>/src/features/onboarding/ui/components/OnboardingQuestion.tsx — renders a question label + free-text textarea input
- [ ] T018 [US1] Create OnboardingForm component in apps/<web-app>/src/features/onboarding/ui/components/OnboardingForm.tsx — 3 questions (areas of interest, geographic preferences, funding focus), submit button, uses OnboardingQuestion for each
- [ ] T019 [US2] Create SkipOnboardingLink component in apps/<web-app>/src/features/onboarding/ui/components/SkipOnboardingLink.tsx — "Skip for now" link/button, clearly visible
- [ ] T020 [US1] Create useOnboarding hook in apps/<web-app>/src/features/onboarding/hooks/useOnboarding.ts — manages form state, submit via API client, handles skip action
- [ ] T021 [US1] Create OnboardingPage in apps/<web-app>/src/features/onboarding/ui/pages/OnboardingPage.tsx — renders OnboardingForm + SkipOnboardingLink, route: /onboarding. Redirects to dashboard on submit or skip
- [ ] T022 [US1] Update registration flow to redirect to /onboarding after role selection (modify role selection success handler in 002-registration-roles)
- [ ] T023 [US1] Write component tests for OnboardingForm in apps/<web-app>/src/features/onboarding/__tests__/OnboardingForm.spec.tsx — fill and submit, validation, skip
- [ ] T024 [P] [US2] Write component test for skip flow in apps/<web-app>/src/features/onboarding/__tests__/SkipOnboarding.spec.tsx — skip redirects to platform, no data saved

**Checkpoint**: US1+US2 complete — onboarding flow works, skip works, preferences saved

---

## Phase 4: US3 — Update Preferences (Priority: P2)

**Goal**: Users can view and update their onboarding answers from account settings. Users who skipped can complete preferences later.

**Independent Test**: Log in → account settings → My Preferences → edit answers → save → verify updated.

- [ ] T025 [US3] Create PreferencesForm component in apps/<web-app>/src/features/auth/ui/components/PreferencesForm.tsx — reuses same fields as OnboardingForm but pre-fills existing values, save button
- [ ] T026 [US3] Create PreferencesPage in apps/<web-app>/src/features/auth/ui/pages/PreferencesPage.tsx — route: /account/preferences, fetches current preferences, renders PreferencesForm
- [ ] T027 [US3] Add "My Preferences" link to account settings navigation
- [ ] T028 [US3] Handle skipped-then-completing flow — when user who skipped saves preferences, set is_onboarding_completed=true
- [ ] T029 [US3] Write component tests for PreferencesPage in apps/<web-app>/src/features/auth/__tests__/PreferencesPage.spec.tsx — load existing, edit, save, empty state for skipped users

**Checkpoint**: US3 complete — preferences editable from account settings

---

## Phase 5: US4 — Personalized Recommendations (Priority: P2)

**Goal**: Homepage shows "Recommended for you" section for users who completed onboarding, based on keyword matching between preferences and project data. Prompt shown for users without preferences.

**Independent Test**: Complete onboarding with specific interests → visit homepage → see recommended projects matching those interests.

### Backend — Recommendations

- [ ] T030 [US4] Create RecommendationsService in apps/<backend-app>/src/modules/preferences/recommendations.service.ts — extract keywords from user's free-text preferences, match against project thematic_areas, location, region, description. Score by match count. Return top N APPROVED projects ordered by score. Fall back to recent projects if < 3 matches
- [ ] T031 [US4] Implement GET /v1/recommendations in PreferencesController — requires auth, accepts limit query param (default 6, max 20), returns recommendations with matchScore and matchReasons. Returns empty with hasPreferences=false if no preferences
- [ ] T032 [US4] Write unit tests for RecommendationsService in apps/<backend-app>/src/modules/preferences/__tests__/recommendations.service.spec.ts — keyword extraction, matching, scoring, no preferences, no matches, fallback

### Frontend — Recommendations Section

- [ ] T033 [US4] Create recommendations feature module scaffold: apps/<web-app>/src/features/recommendations/ with ui/components/, hooks/, index.ts
- [ ] T034 [US4] Create useRecommendations hook in apps/<web-app>/src/features/recommendations/hooks/useRecommendations.ts — fetches recommendations via TanStack Query, only fires when user is authenticated
- [ ] T035 [US4] Create RecommendedSection component in apps/<web-app>/src/features/recommendations/ui/components/RecommendedSection.tsx — renders recommended project cards (reuses ProjectCard), shows matchReasons below each card
- [ ] T036 [US4] Create CompletePreferencesPrompt component in apps/<web-app>/src/features/recommendations/ui/components/CompletePreferencesPrompt.tsx — card prompting user to complete preferences with link to /onboarding or /account/preferences
- [ ] T037 [US4] Integrate RecommendedSection into HomePage — show above trending section for authenticated users. Show CompletePreferencesPrompt if no preferences. Hide entirely for unauthenticated visitors
- [ ] T038 [US4] Write component tests for RecommendedSection in apps/<web-app>/src/features/recommendations/__tests__/RecommendedSection.spec.tsx — with recommendations, empty, no preferences prompt
- [ ] T039 [P] [US4] Write integration test for GET /v1/recommendations in apps/<backend-app>/src/modules/preferences/__tests__/recommendations.integration.spec.ts

**Checkpoint**: US4 complete — personalized recommendations visible on homepage

---

## Phase 6: Polish & Cross-Cutting Concerns

- [ ] T040 [P] Add responsive layout for OnboardingPage and PreferencesPage — mobile 375px, tablet 768px, desktop 1440px
- [ ] T041 [P] Write E2E test for onboarding flow in tests/e2e/onboarding/onboarding-flow.spec.ts — complete flow from registration to recommendations
- [ ] T042 [P] Write E2E test for skip onboarding in tests/e2e/onboarding/skip-onboarding.spec.ts — skip, verify platform works, no recommendations
- [ ] T043 [P] Write E2E test for update preferences in tests/e2e/onboarding/update-preferences.spec.ts — account settings, edit, save, verify updated recommendations
- [ ] T044 [P] Write E2E test for recommendations in tests/e2e/onboarding/recommendations.spec.ts — verify matching logic end-to-end
- [ ] T045 Run quickstart.md validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies
- **Phase 2 (Foundation)**: Depends on Phase 1
- **Phase 3 (US1+US2)**: Depends on Phase 2 — MVP onboarding flow
- **Phase 4 (US3)**: Depends on Phase 3 (extends preferences with edit)
- **Phase 5 (US4)**: Depends on Phase 3 (needs saved preferences to generate recommendations). Can run in parallel with Phase 4
- **Phase 6 (Polish)**: Depends on all stories

### User Story Dependencies

```
Phase 1 (Setup)
    │
Phase 2 (Foundation: DB + API client)
    │
Phase 3 (US1+US2: Onboarding + Skip) ─── MVP
    │
    ├──────────────┐
    │              │
Phase 4          Phase 5
US3 Update       US4 Recommendations
Preferences      (parallel with Phase 4)
    │              │
    └──────────────┘
           │
       Phase 6 (Polish + E2E)
```

### Parallel Opportunities

- **Phase 1**: T001+T002 types in parallel
- **Phase 2**: T004 (migration) + T005 (API client) in parallel
- **Phase 3**: Backend (T006-T015) and frontend (T016-T024) overlap after DTOs
- **Phases 4+5**: Can run in parallel
- **Phase 6**: All E2E tests (T041-T044) in parallel

---

## Implementation Strategy

### MVP First (Phase 1 + 2 + 3)

1. Phase 1: Shared types
2. Phase 2: DB schema + API client
3. Phase 3: Onboarding form + skip flow
4. **STOP and VALIDATE**: User completes or skips onboarding after registration
5. Deploy/demo

### Incremental Delivery

1. **Phase 1+2+3** → Onboarding works (MVP)
2. **Phase 4** → Edit preferences from settings → Deploy
3. **Phase 5** → Recommendations on homepage → Deploy
4. **Phase 6** → E2E + polish → Final release

---

## Notes

- US1+US2 grouped — complete and skip are two paths of the same flow
- Preferences stored as free text — enables future AI matching (US-31)
- Recommendations use simple keyword matching initially, not AI embeddings
- OnboardingForm reused in PreferencesPage (same fields, different context)
- Recommendations only shown to authenticated users — hidden for visitors
- Onboarding is always skippable — no feature gated behind it (FR-004)
