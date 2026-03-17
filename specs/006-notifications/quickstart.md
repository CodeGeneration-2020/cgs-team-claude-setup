# Quickstart: Notifications

**Feature**: 006-notifications | **Date**: 2026-03-17

## Prerequisites

- Features 001-005 implemented (homepage, registration, projects, bookmarks, onboarding)
- Email service configured

## Setup

```bash
pnpm install
pnpm --filter <backend-app> prisma generate
pnpm --filter <backend-app> prisma migrate dev    # adds notifications table, notification_processor_state, email pref field
```

## Testing

```bash
pnpm --filter <backend-app> test -- --testPathPattern=notifications
pnpm --filter <web-app> test -- --testPathPattern=notifications
pnpm --filter <web-app> test:e2e -- --testPathPattern=notifications
```

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | /v1/notifications | FUNDER | List notifications (paginated) |
| GET | /v1/notifications/unread-count | FUNDER | Unread badge count |
| POST | /v1/notifications/:id/read | FUNDER | Mark one as read |
| POST | /v1/notifications/read-all | FUNDER | Mark all as read |
| GET | /v1/notifications/settings | Authenticated | Get email preference |
| PATCH | /v1/notifications/settings | Authenticated | Update email preference |

## Key Modules

**Backend:** `notifications` module with 3 services:
- NotificationsService — CRUD, list, read/unread
- NotificationGeneratorService — background job scanning audit log → creating notifications
- NotificationEmailService — email delivery with retry

**Frontend:** `notifications` feature module (center page, badge, settings panel)
**Shared:** notification types + event type enum in shared-types
