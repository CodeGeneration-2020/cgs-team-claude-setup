# Implementation Plan: Notifications

**Branch**: `006-notifications` | **Date**: 2026-03-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/006-notifications/spec.md`

## Summary

Add an in-app Notification Center and optional email notifications for Funders. Notifications are generated from project audit log events (bookmark project changes) and recommendation matches (new projects matching onboarding profile). Users can view, mark as read, and manage notifications. Email notifications mirror in-app events with an opt-out toggle in account settings. A background notification generator processes audit log events and creates notification records.

## Technical Context

**Language/Version**: TypeScript (strict mode)
**Primary Dependencies**: NestJS (backend), React.js + Tailwind CSS (frontend), Zustand + TanStack Query (state/data), Prisma ORM
**Storage**: PostgreSQL (notifications table, notification_preferences on user)
**Testing**: Jest (unit + integration), Playwright (E2E), React Testing Library
**Target Platform**: Web application (desktop + tablet + mobile responsive)
**Project Type**: Web service (monorepo)
**Performance Goals**: Notifications generated within 5 minutes of event, Notification Center loads < 2 seconds
**Constraints**: Email opt-out must not affect in-app, batching for rapid changes, no real-time WebSocket (poll-based)
**Scale/Scope**: Thousands of notifications per user over time, email delivery via existing email service

## Constitution Check

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Monorepo-First | PASS | Adds to existing apps |
| II. Clean Architecture & SOLID | PASS | NotificationService generates, controller serves, email service separate |
| III. Modular Architecture | PASS | Backend: `notifications` module. Frontend: `notifications` feature module |
| IV. Strict Type Safety | PASS | Notification types in shared-types |
| V. Security by Design | PASS | @Roles(FUNDER) on notification endpoints, user can only see own notifications |
| VI. Testing Discipline | PASS | Unit + integration + E2E |
| VII. Independent Deployability | PASS | No new apps |
| VIII. Observability-First | PASS | Notification events logged via Pino |
| IX. Shared-Before-Custom | PASS | Notification types in shared-types |
| X. Design Token Management | PASS | Badge, read/unread styling via tokens |

**Gate Result: PASS**

## Project Structure

### Documentation

```text
specs/006-notifications/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── notifications.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code

```text
# Backend
apps/<backend-app>/src/modules/
├── notifications/
│   ├── notifications.module.ts
│   ├── notifications.controller.ts
│   ├── notifications.service.ts
│   ├── notification-generator.service.ts    # Background job: audit log → notifications
│   ├── notification-email.service.ts        # Email delivery
│   ├── dto/
│   │   ├── notification-response.dto.ts
│   │   ├── notifications-query.dto.ts
│   │   └── update-preferences.dto.ts
│   ├── entities/
│   │   └── notification.entity.ts
│   ├── __tests__/
│   └── index.ts

# Frontend
apps/<web-app>/src/features/
├── notifications/
│   ├── ui/
│   │   ├── pages/
│   │   │   └── NotificationCenterPage.tsx
│   │   └── components/
│   │       ├── NotificationList.tsx
│   │       ├── NotificationItem.tsx
│   │       ├── NotificationBadge.tsx
│   │       └── NotificationSettingsPanel.tsx
│   ├── state/
│   │   └── useNotificationStore.ts
│   ├── hooks/
│   │   ├── useNotifications.ts
│   │   ├── useUnreadCount.ts
│   │   └── useNotificationSettings.ts
│   ├── __tests__/
│   └── index.ts

# Shared
packages/shared-types/src/
├── notification/
│   ├── notification.ts
│   ├── notification-event-type.ts
│   ├── notification-preferences.ts
│   └── index.ts

# E2E
tests/e2e/notifications/
├── notification-center.spec.ts
├── mark-read.spec.ts
├── email-settings.spec.ts
└── notification-generation.spec.ts
```

**Structure Decision**: Backend `notifications` module with three services: NotificationsService (CRUD/query), NotificationGeneratorService (background processing of audit events into notifications), NotificationEmailService (email sending). Frontend `notifications` feature module for the center page and navigation badge. Follows Constitution Principles III, IX.

## Complexity Tracking

No violations.
