# Quickstart: AI Assistant

**Feature**: 007-ai-assistant | **Date**: 2026-03-17

## Prerequisites

- Features 001-005 implemented (homepage search, registration, projects, bookmarks, onboarding)
- LLM API key configured (for contextual responses)

## Environment Variables

```bash
APP_LLM_API_KEY=<your-llm-api-key>
APP_LLM_MODEL=<model-name>
```

## Setup

```bash
pnpm install
pnpm --filter <backend-app> prisma generate
pnpm --filter <backend-app> prisma migrate dev    # adds assistant_questions, assistant_sessions tables
pnpm --filter <backend-app> prisma db seed         # seeds predefined questions
```

## Testing

```bash
pnpm --filter <backend-app> test -- --testPathPattern=assistant
pnpm --filter <web-app> test -- --testPathPattern=ai-assistant
pnpm --filter <web-app> test:e2e -- --testPathPattern=ai-assistant
```

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /v1/assistant/start | Optional | Start session, get first question |
| POST | /v1/assistant/:sessionId/answer | Optional | Submit answer, get next question/results |
| POST | /v1/assistant/:sessionId/skip | Optional | Skip current question |
| GET | /v1/assistant/questions | Public | List predefined questions |
| POST | /v1/assistant/:sessionId/transfer | Required | Transfer visitor session to user |

## Key Modules

**Backend:** `assistant` module with AssistantService (flow) + AssistantLlmService (LLM calls)
**Frontend:** `ai-assistant` feature module (widget, chat, results)
**Shared:** assistant types in shared-types
