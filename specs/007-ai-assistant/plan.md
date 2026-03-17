# Implementation Plan: AI Assistant

**Branch**: `007-ai-assistant` | **Date**: 2026-03-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/007-ai-assistant/spec.md`

## Summary

Add an AI-powered chat assistant widget on the homepage that guides visitors through a predefined question sequence to understand their interests. The assistant generates contextual responses using an LLM and matches answers to project search results. Questions are data-driven (configurable without code changes). Authenticated users' answers are persisted and merged into onboarding preferences. Visitors get session-only results.

## Technical Context

**Language/Version**: TypeScript (strict mode)
**Primary Dependencies**: NestJS (backend), React.js + Tailwind CSS (frontend), Zustand + TanStack Query (state/data), Prisma ORM, LLM API (for contextual responses and answer-to-query matching)
**Storage**: PostgreSQL (assistant_questions, assistant_sessions for authenticated users)
**Testing**: Jest (unit + integration), Playwright (E2E), React Testing Library
**Target Platform**: Web application (desktop + tablet + mobile responsive)
**Project Type**: Web service (monorepo)
**Performance Goals**: Responses within 3 seconds, widget opens instantly
**Constraints**: Works for unauthenticated visitors (no login required), predefined questions configurable via data, LLM usage governed by AI governance principles
**Scale/Scope**: Concurrent assistant sessions, 3-5 questions per flow

## Constitution Check

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Monorepo-First | PASS | Adds to existing apps |
| II. Clean Architecture & SOLID | PASS | AssistantService handles flow logic, LLM calls in infrastructure layer |
| III. Modular Architecture | PASS | Backend: `assistant` module. Frontend: `ai-assistant` feature module |
| IV. Strict Type Safety | PASS | Assistant types in shared-types |
| V. Security by Design | PASS | Public endpoint for visitors, AI requests exclude PII (FR from AI Governance US-35) |
| VI. Testing Discipline | PASS | Unit + integration + E2E |
| VII. Independent Deployability | PASS | No new apps |
| VIII. Observability-First | PASS | AI usage logged per request (AI Governance US-36) |
| IX. Shared-Before-Custom | PASS | Reuses ProjectCard, search results display |
| X. Design Token Management | PASS | Chat UI styled via design tokens |

**Gate Result: PASS**

## Project Structure

### Documentation

```text
specs/007-ai-assistant/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── assistant.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code

```text
# Backend
apps/<backend-app>/src/modules/
├── assistant/
│   ├── assistant.module.ts
│   ├── assistant.controller.ts
│   ├── assistant.service.ts
│   ├── assistant-llm.service.ts          # LLM integration for contextual responses
│   ├── dto/
│   │   ├── start-session.dto.ts
│   │   ├── submit-answer.dto.ts
│   │   ├── assistant-message.dto.ts
│   │   └── session-results.dto.ts
│   ├── entities/
│   │   ├── assistant-question.entity.ts
│   │   └── assistant-session.entity.ts
│   ├── __tests__/
│   └── index.ts

# Frontend
apps/<web-app>/src/features/
├── ai-assistant/
│   ├── ui/
│   │   └── components/
│   │       ├── AssistantWidget.tsx        # Floating button + panel
│   │       ├── AssistantChat.tsx          # Chat message flow
│   │       ├── ChatMessage.tsx            # Single message bubble
│   │       ├── ChatInput.tsx              # Free-text input
│   │       ├── SkipButton.tsx
│   │       └── AssistantResults.tsx       # Results after flow
│   ├── state/
│   │   └── useAssistantStore.ts
│   ├── hooks/
│   │   ├── useAssistant.ts
│   │   └── useAssistantSession.ts
│   ├── __tests__/
│   └── index.ts

# Shared
packages/shared-types/src/
├── assistant/
│   ├── assistant-question.ts
│   ├── assistant-message.ts
│   ├── assistant-session.ts
│   └── index.ts

# Seed
prisma/
├── seed-assistant-questions.ts

# E2E
tests/e2e/ai-assistant/
├── assistant-flow.spec.ts
├── skip-close.spec.ts
└── results.spec.ts
```

**Structure Decision**: Backend `assistant` module with two services: AssistantService (session/flow management) and AssistantLlmService (LLM calls for contextual responses). Frontend `ai-assistant` feature module with chat widget components. Questions seeded via database. Follows Constitution Principles III, V, VIII.

## Complexity Tracking

No violations.
