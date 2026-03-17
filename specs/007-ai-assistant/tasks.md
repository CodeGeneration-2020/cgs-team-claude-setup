# Tasks: AI Assistant

**Input**: Design documents from `/specs/007-ai-assistant/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: TDD — constitution requires 80% coverage.

**Organization**: US1 (Guided Flow) + US3 (View Results) grouped as P1 MVP — the flow is pointless without results. US2 (Store Answers) is P2.

## Format: `[ID] [P?] [Story] Description`

## Path Conventions

- **Backend**: `apps/<backend-app>/src/modules/`
- **Frontend**: `apps/<web-app>/src/features/`
- **Shared types**: `packages/shared-types/src/`
- **API client**: `packages/api-client/src/`
- **E2E tests**: `tests/e2e/ai-assistant/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Shared types and API client

- [ ] T001 Add assistant types in packages/shared-types/src/assistant/assistant-question.ts — AssistantQuestion with id, text, displayOrder, isSkippable, placeholderText
- [ ] T002 [P] Add assistant message/session types in packages/shared-types/src/assistant/assistant-message.ts, assistant-session.ts — AnswerSubmission, AssistantResponse (contextualResponse, question, isComplete, searchQuery, redirectUrl)
- [ ] T003 Re-export from packages/shared-types/src/assistant/index.ts and packages/shared-types/src/index.ts

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Database schema, question seed, LLM service, API client

- [ ] T004 Create Prisma migration for assistant_questions table: id (CUID), question_text (String max 500), display_order (Int unique), is_skippable (Boolean default true), placeholder_text (String nullable max 200), is_active (Boolean default true), created_at. Add index on display_order
- [ ] T005 [P] Create Prisma migration for assistant_sessions table: id (CUID), user_id (FK nullable, SET NULL), answers (JSON), extracted_query (String nullable), is_completed (Boolean default false), started_at, completed_at (nullable). Add index on user_id
- [ ] T006 Create seed script for predefined assistant questions in prisma/seed-assistant-questions.ts — seed 4-5 questions: interests, regions, project types, funding focus, implementation stage
- [ ] T007 [P] Add assistant API client methods in packages/api-client/src/assistant/ — startSession(), submitAnswer(), skipQuestion(), getQuestions(), transferSession()

**Checkpoint**: Foundation ready

---

## Phase 3: US1 + US3 — Guided Flow + View Results (Priority: P1) MVP

**Goal**: Visitor opens the assistant widget on the homepage, goes through a predefined question sequence with contextual AI responses, and at the end sees project search results matching their answers. Works without login.

**Independent Test**: Open homepage → click assistant widget → answer questions → see contextual responses → flow completes → redirected to search results matching answers.

### Backend — Assistant Module

- [ ] T008 [US1] Create assistant module scaffold: apps/<backend-app>/src/modules/assistant/assistant.module.ts, assistant.controller.ts, assistant.service.ts, dto/, entities/, index.ts
- [ ] T009 [US1] Create AssistantQuestion entity in apps/<backend-app>/src/modules/assistant/entities/assistant-question.entity.ts
- [ ] T010 [P] [US1] Create AssistantSession entity in apps/<backend-app>/src/modules/assistant/entities/assistant-session.entity.ts
- [ ] T011 [P] [US1] Create StartSessionDto (empty, returns first question) and SubmitAnswerDto (questionId, answer max 2000) in apps/<backend-app>/src/modules/assistant/dto/
- [ ] T012 [P] [US1] Create AssistantMessageDto (contextualResponse, question, isComplete, searchQuery, redirectUrl) in apps/<backend-app>/src/modules/assistant/dto/assistant-message.dto.ts
- [ ] T013 [US1] Create AssistantLlmService in apps/<backend-app>/src/modules/assistant/assistant-llm.service.ts — generates contextual response given current question + previous Q&A pairs. Extracts search query from all answers on final question. Logs token usage and latency. Excludes PII from LLM requests
- [ ] T014 [US1] Implement AssistantService.startSession() in apps/<backend-app>/src/modules/assistant/assistant.service.ts — create session record, load first active question by display_order, return session ID + question
- [ ] T015 [US1] Implement AssistantService.submitAnswer() — save answer to session, call LLM for contextual response, determine next question or complete flow. On completion: extract search query via LLM, set is_completed=true
- [ ] T016 [US1] Implement AssistantService.skipQuestion() — move to next question without saving an answer, LLM generates brief skip acknowledgment
- [ ] T017 [US1] Implement POST /v1/assistant/start in AssistantController — @Public()
- [ ] T018 [US1] Implement POST /v1/assistant/:sessionId/answer in AssistantController — @Public()
- [ ] T019 [US1] Implement POST /v1/assistant/:sessionId/skip in AssistantController — @Public()
- [ ] T020 [US1] Implement GET /v1/assistant/questions in AssistantController — @Public(), returns active questions
- [ ] T021 [US1] Write unit tests for AssistantService in apps/<backend-app>/src/modules/assistant/__tests__/assistant.service.spec.ts — start, answer, skip, complete, session management
- [ ] T022 [P] [US1] Write unit tests for AssistantLlmService in apps/<backend-app>/src/modules/assistant/__tests__/assistant-llm.spec.ts — contextual response generation, query extraction, PII exclusion, error handling
- [ ] T023 [US3] Write integration tests for assistant endpoints in apps/<backend-app>/src/modules/assistant/__tests__/assistant.integration.spec.ts — full flow from start to results

### Frontend — AI Assistant Widget

- [ ] T024 [US1] Create ai-assistant feature module scaffold: apps/<web-app>/src/features/ai-assistant/ with ui/components/, state/, hooks/, index.ts
- [ ] T025 [US1] Create useAssistantStore (Zustand) in apps/<web-app>/src/features/ai-assistant/state/useAssistantStore.ts — isOpen, sessionId, messages (array of bot/user messages), currentQuestion, isComplete, searchQuery
- [ ] T026 [US1] Create useAssistantSession hook in apps/<web-app>/src/features/ai-assistant/hooks/useAssistantSession.ts — manages start, answer, skip mutations via TanStack Query, updates store
- [ ] T027 [US1] Create ChatMessage component in apps/<web-app>/src/features/ai-assistant/ui/components/ChatMessage.tsx — renders a single message bubble (bot or user), different styling for each
- [ ] T028 [US1] Create ChatInput component in apps/<web-app>/src/features/ai-assistant/ui/components/ChatInput.tsx — free-text input with send button, placeholder from current question
- [ ] T029 [US1] Create SkipButton component in apps/<web-app>/src/features/ai-assistant/ui/components/SkipButton.tsx — "Skip this question" link, visible when question.isSkippable
- [ ] T030 [US1] Create AssistantChat component in apps/<web-app>/src/features/ai-assistant/ui/components/AssistantChat.tsx — renders message list, ChatInput, SkipButton, loading indicator during LLM response, auto-scrolls to latest message
- [ ] T031 [US3] Create AssistantResults component in apps/<web-app>/src/features/ai-assistant/ui/components/AssistantResults.tsx — shown after flow completes: "View Results" button that navigates to /search?q={extractedQuery}&source=assistant
- [ ] T032 [US1] Create AssistantWidget component in apps/<web-app>/src/features/ai-assistant/ui/components/AssistantWidget.tsx — FAB button (bottom-right), opens chat panel (side drawer on desktop, full-screen overlay on mobile), close button, minimizable
- [ ] T033 [US1] Integrate AssistantWidget into HomePage — render the FAB on the homepage, widget initializes session on open
- [ ] T034 [US1] Implement AI service unavailable fallback — if LLM call fails, show: "The assistant is temporarily unavailable. Please use the search bar to find projects."
- [ ] T035 [US1] Write component tests for AssistantChat in apps/<web-app>/src/features/ai-assistant/__tests__/AssistantChat.spec.tsx — message rendering, input, skip, loading
- [ ] T036 [P] [US1] Write component tests for AssistantWidget in apps/<web-app>/src/features/ai-assistant/__tests__/AssistantWidget.spec.tsx — open, close, minimize, mobile overlay
- [ ] T037 [P] [US3] Write component tests for AssistantResults in apps/<web-app>/src/features/ai-assistant/__tests__/AssistantResults.spec.tsx — redirect to search

**Checkpoint**: US1+US3 complete — visitor can use assistant, get contextual responses, see matching results

---

## Phase 4: US2 — Store and Reuse Answers (Priority: P2)

**Goal**: Authenticated users' assistant answers are persisted to their profile and merged with onboarding preferences. Visitors' answers transfer on login/register.

**Independent Test**: Log in → complete assistant → log out → log back in → verify preferences updated. As visitor: complete assistant → register → verify answers transferred.

### Backend

- [ ] T038 [US2] Implement session persistence for authenticated users — when user_id is present on session, save to DB and merge extracted answers into UserPreferences (from 005-onboarding)
- [ ] T039 [US2] Implement POST /v1/assistant/:sessionId/transfer in AssistantController — requires auth, links visitor session to authenticated user, merges into preferences
- [ ] T040 [US2] Write unit tests for answer persistence and transfer in apps/<backend-app>/src/modules/assistant/__tests__/assistant-persistence.spec.ts

### Frontend

- [ ] T041 [US2] After login/register, check for active assistant session in store → if exists, call transfer endpoint to link to new account
- [ ] T042 [US2] Write component test for session transfer flow in apps/<web-app>/src/features/ai-assistant/__tests__/SessionTransfer.spec.tsx

**Checkpoint**: US2 complete — answers persisted for auth users, transferred on registration

---

## Phase 5: Polish & Cross-Cutting Concerns

- [ ] T043 [P] Add responsive layout for AssistantWidget — desktop: 400px side panel, mobile: full-screen overlay with back button
- [ ] T044 [P] Add chat panel persistence — minimize and re-open without losing conversation progress during the session
- [ ] T045 [P] Write E2E test for full assistant flow in tests/e2e/ai-assistant/assistant-flow.spec.ts — open, answer all questions, see results
- [ ] T046 [P] Write E2E test for skip and close in tests/e2e/ai-assistant/skip-close.spec.ts — skip questions, close mid-flow
- [ ] T047 [P] Write E2E test for results integration in tests/e2e/ai-assistant/results.spec.ts — verify search results match assistant answers
- [ ] T048 Run quickstart.md validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies
- **Phase 2 (Foundation)**: Depends on Phase 1
- **Phase 3 (US1+US3)**: Depends on Phase 2 — MVP assistant flow + results
- **Phase 4 (US2)**: Depends on Phase 3 (extends with persistence)
- **Phase 5 (Polish)**: Depends on all stories

### User Story Dependencies

```
Phase 1 (Setup)
    │
Phase 2 (Foundation: DB + seeds + LLM service + API client)
    │
Phase 3 (US1+US3: Guided Flow + Results) ─── MVP
    │
Phase 4 (US2: Answer Persistence)
    │
Phase 5 (Polish + E2E)
```

### Parallel Opportunities

- **Phase 2**: T004+T005 migrations in parallel, T006+T007 seed+client in parallel
- **Phase 3**: Backend (T008-T023) and frontend (T024-T037) overlap after DTOs defined
- **Phase 3**: T021+T022 service tests in parallel, T035+T036+T037 component tests in parallel
- **Phase 5**: All E2E tests (T045-T047) in parallel

---

## Implementation Strategy

### MVP First (Phase 1 + 2 + 3)

1. Phase 1: Shared types
2. Phase 2: DB, question seeds, LLM service, API client
3. Phase 3: Widget + chat flow + contextual responses + results
4. **STOP and VALIDATE**: Visitor opens assistant, answers questions, sees matching projects
5. Deploy/demo

### Incremental Delivery

1. **Phase 1+2+3** → Assistant works for visitors (MVP)
2. **Phase 4** → Answer persistence for registered users → Deploy
3. **Phase 5** → E2E + polish → Final release

---

## Notes

- US1+US3 grouped — guided flow without results is pointless
- LLM calls exclude PII (AI Governance US-35), usage logged (US-36)
- Questions stored in DB, seeded — configurable without code changes
- Results redirect to existing search page (001-homepage) — no duplicate UI
- Session transfer on login handles visitor→user transition
- FAB widget: bottom-right, desktop side panel 400px, mobile full-screen
