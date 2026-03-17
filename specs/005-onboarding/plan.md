# Implementation Plan: Onboarding

**Branch**: `005-onboarding` | **Date**: 2026-03-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/005-onboarding/spec.md`

## Summary

Add a post-registration onboarding questionnaire where users describe their interests, geographic preferences, and funding/project focus. Answers are stored in a UserPreferences entity and used to generate a "Recommended for you" section on the homepage. Users can skip onboarding, update preferences from account settings, and the system matches preferences against project taxonomy fields for recommendations.

## Technical Context

**Language/Version**: TypeScript (strict mode)
**Primary Dependencies**: NestJS (backend), React.js + Tailwind CSS (frontend), Zustand + TanStack Query (state/data), Prisma ORM
**Storage**: PostgreSQL (user_preferences table)
**Testing**: Jest (unit + integration), Playwright (E2E), React Testing Library
**Target Platform**: Web application (desktop + tablet + mobile responsive)
**Project Type**: Web service (monorepo)
**Performance Goals**: Onboarding form submit < 2 seconds, recommendations load < 2 seconds
**Constraints**: Onboarding must be skippable, preferences are role-agnostic, initial matching is simple field-based (not AI)
**Scale/Scope**: All registered users, recommendations from hundreds/thousands of projects

## Constitution Check

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Monorepo-First | PASS | Adds to existing apps |
| II. Clean Architecture & SOLID | PASS | Preferences logic in service, recommendation logic in service |
| III. Modular Architecture | PASS | Backend: `preferences` module. Frontend: `onboarding` feature module + preferences in account settings |
| IV. Strict Type Safety | PASS | Preference types in shared-types |
| V. Security by Design | PASS | Preferences behind auth guard |
| VI. Testing Discipline | PASS | Unit + integration + E2E |
| VII. Independent Deployability | PASS | No new apps |
| VIII. Observability-First | PASS | Structured logging |
| IX. Shared-Before-Custom | PASS | Preference types in shared-types |
| X. Design Token Management | PASS | Form styling via tokens |

**Gate Result: PASS**

## Project Structure

### Documentation

```text
specs/005-onboarding/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── preferences.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code

```text
# Backend
apps/<backend-app>/src/modules/
├── preferences/
│   ├── preferences.module.ts
│   ├── preferences.controller.ts
│   ├── preferences.service.ts
│   ├── recommendations.service.ts
│   ├── dto/
│   │   ├── save-preferences.dto.ts
│   │   └── preferences-response.dto.ts
│   ├── entities/
│   │   └── user-preferences.entity.ts
│   ├── __tests__/
│   └── index.ts

# Frontend
apps/<web-app>/src/features/
├── onboarding/
│   ├── ui/
│   │   ├── pages/
│   │   │   └── OnboardingPage.tsx
│   │   └── components/
│   │       ├── OnboardingForm.tsx
│   │       ├── OnboardingQuestion.tsx
│   │       └── SkipOnboardingLink.tsx
│   ├── hooks/
│   │   └── useOnboarding.ts
│   ├── __tests__/
│   └── index.ts
├── recommendations/
│   ├── ui/
│   │   └── components/
│   │       ├── RecommendedSection.tsx
│   │       └── CompletePreferencesPrompt.tsx
│   ├── hooks/
│   │   └── useRecommendations.ts
│   ├── __tests__/
│   └── index.ts

# Preferences in account settings (extends existing auth feature)
apps/<web-app>/src/features/auth/
├── ui/
│   ├── pages/
│   │   └── PreferencesPage.tsx        # new
│   └── components/
│       └── PreferencesForm.tsx         # new (reuses OnboardingForm)

# Shared
packages/shared-types/src/
├── preferences/
│   ├── user-preferences.ts
│   ├── recommendation.ts
│   └── index.ts

# E2E
tests/e2e/onboarding/
├── onboarding-flow.spec.ts
├── skip-onboarding.spec.ts
├── update-preferences.spec.ts
└── recommendations.spec.ts
```

**Structure Decision**: Backend `preferences` module handles both onboarding data and recommendation generation. Frontend has `onboarding` (post-registration flow) and `recommendations` (homepage section) as separate modules. Preferences editing lives in the existing auth/account feature. Follows Constitution Principles III, IX.

## Complexity Tracking

No violations.
