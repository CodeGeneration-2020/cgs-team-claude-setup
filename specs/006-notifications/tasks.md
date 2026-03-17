# Tasks: Notifications

**Input**: Design documents from `/specs/006-notifications/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: TDD — constitution requires 80% coverage.

**Organization**: US1 (Notification Center) + US2 (Mark as Read) grouped as P1 MVP — the center is useless without read tracking. US3 (Email) + US4 (Opt-out) grouped as P2 since they're tightly coupled.

## Format: `[ID] [P?] [Story] Description`

## Path Conventions

- **Backend**: `apps/<backend-app>/src/modules/`
- **Frontend**: `apps/<web-app>/src/features/`
- **Shared types**: `packages/shared-types/src/`
- **API client**: `packages/api-client/src/`
- **E2E tests**: `tests/e2e/notifications/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Shared types and API client

- [ ] T001 Add NotificationEventType enum in packages/shared-types/src/notification/notification-event-type.ts — BOOKMARK_PROJECT_UPDATED, BOOKMARK_PROJECT_STATUS_CHANGED, NEW_MATCHING_PROJECT, PROJECT_NO_LONGER_AVAILABLE
- [ ] T002 [P] Add notification types in packages/shared-types/src/notification/notification.ts — NotificationResponse shape with id, eventType, title, message, projectId, projectTitle, isRead, changeCount, createdAt, readAt
- [ ] T003 [P] Add notification preferences type in packages/shared-types/src/notification/notification-preferences.ts — isEmailNotificationsEnabled
- [ ] T004 Re-export from packages/shared-types/src/notification/index.ts and packages/shared-types/src/index.ts

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Database schema, background job infrastructure, API client

- [ ] T005 Create Prisma migration for notifications table: id (CUID), user_id (FK), project_id (FK, SET NULL on delete, nullable), event_type (NotificationEventType enum), title (String max 200), message (String max 1000), is_read (Boolean default false), change_count (Int default 1), source_audit_id (String nullable), created_at, read_at (nullable). Add indexes on (user_id, created_at DESC), (user_id, is_read), (user_id, project_id, created_at)
- [ ] T006 [P] Create Prisma migration for notification_processor_state table: id (String PK), last_processed_at (DateTime), updated_at. Seed initial row with id="notification_processor"
- [ ] T007 [P] Create Prisma migration to add is_email_notifications_enabled (Boolean default true) to users or user_preferences table
- [ ] T008 Add notification API client methods in packages/api-client/src/notifications/ — getNotifications(), getUnreadCount(), markRead(), markAllRead(), getNotificationSettings(), updateNotificationSettings()

**Checkpoint**: Foundation ready

---

## Phase 3: US1 + US2 — Notification Center + Mark as Read (Priority: P1) MVP

**Goal**: Funders see a Notification Center with paginated notifications from bookmark changes and matching new projects. Unread badge in nav. Notifications can be marked as read individually or all at once.

**Independent Test**: Bookmark a project → project gets updated → open Notification Center → see notification → mark as read → badge decreases → click notification → navigate to project.

### Backend — Notifications Module

- [ ] T009 [US1] Create notifications module scaffold: apps/<backend-app>/src/modules/notifications/notifications.module.ts, notifications.controller.ts, notifications.service.ts, dto/, entities/, index.ts
- [ ] T010 [US1] Create Notification entity in apps/<backend-app>/src/modules/notifications/entities/notification.entity.ts
- [ ] T011 [P] [US1] Create NotificationsQueryDto in apps/<backend-app>/src/modules/notifications/dto/notifications-query.dto.ts — page, pageSize, unreadOnly
- [ ] T012 [P] [US1] Create NotificationResponseDto in apps/<backend-app>/src/modules/notifications/dto/notification-response.dto.ts
- [ ] T013 [US1] Implement NotificationsService.getNotifications() in apps/<backend-app>/src/modules/notifications/notifications.service.ts — query user's notifications with LEFT JOIN to projects, paginated, most recent first, include unreadCount in meta
- [ ] T014 [US1] Implement NotificationsService.getUnreadCount() — lightweight count query for badge
- [ ] T015 [US2] Implement NotificationsService.markRead() — set is_read=true, read_at=now for a single notification (validate ownership)
- [ ] T016 [US2] Implement NotificationsService.markAllRead() — bulk update all unread notifications for user
- [ ] T017 [US1] Create NotificationGeneratorService in apps/<backend-app>/src/modules/notifications/notification-generator.service.ts — scheduled job running every 2-3 minutes: scan project_audit_log for new events since last_processed_at, find Funders who bookmarked affected projects, create notifications with batching (10-min window check)
- [ ] T018 [US1] Implement new-project matching in NotificationGeneratorService — on new APPROVED projects, check Funders with preferences for keyword matches, create NEW_MATCHING_PROJECT notifications for high-confidence matches
- [ ] T019 [US1] Implement GET /v1/notifications in NotificationsController — @Roles(FUNDER, SUPER_ADMIN)
- [ ] T020 [US1] Implement GET /v1/notifications/unread-count in NotificationsController — @Roles(FUNDER, SUPER_ADMIN)
- [ ] T021 [US2] Implement POST /v1/notifications/:id/read in NotificationsController — validates notification belongs to user
- [ ] T022 [US2] Implement POST /v1/notifications/read-all in NotificationsController
- [ ] T023 [US1] Write unit tests for NotificationsService in apps/<backend-app>/src/modules/notifications/__tests__/notifications.service.spec.ts — list, count, pagination, mark read, mark all
- [ ] T024 [P] [US1] Write unit tests for NotificationGeneratorService in apps/<backend-app>/src/modules/notifications/__tests__/notification-generator.spec.ts — audit log scanning, batching, new project matching
- [ ] T025 [US1] Write integration tests for notification endpoints in apps/<backend-app>/src/modules/notifications/__tests__/notifications.integration.spec.ts

### Frontend — Notifications Feature Module

- [ ] T026 [US1] Create notifications feature module scaffold: apps/<web-app>/src/features/notifications/ with ui/pages/, ui/components/, state/, hooks/, index.ts
- [ ] T027 [US1] Create useNotificationStore (Zustand) in apps/<web-app>/src/features/notifications/state/useNotificationStore.ts — unreadCount, notifications list cache
- [ ] T028 [US1] Create useUnreadCount hook in apps/<web-app>/src/features/notifications/hooks/useUnreadCount.ts — polls GET /v1/notifications/unread-count every 60 seconds via TanStack Query
- [ ] T029 [US1] Create NotificationBadge component in apps/<web-app>/src/features/notifications/ui/components/NotificationBadge.tsx — bell icon + unread count badge, placed in global navigation. Hides count when zero
- [ ] T030 [US1] Create useNotifications hook in apps/<web-app>/src/features/notifications/hooks/useNotifications.ts — fetches paginated notifications via TanStack Query
- [ ] T031 [US1] Create NotificationItem component in apps/<web-app>/src/features/notifications/ui/components/NotificationItem.tsx — renders single notification with read/unread styling, click-to-navigate, mark-as-read button
- [ ] T032 [US1] Create NotificationList component in apps/<web-app>/src/features/notifications/ui/components/NotificationList.tsx — renders list of NotificationItems, pagination, empty state, "Mark all as read" button
- [ ] T033 [US1] Create NotificationCenterPage in apps/<web-app>/src/features/notifications/ui/pages/NotificationCenterPage.tsx — route: /notifications (FUNDER only), renders NotificationList
- [ ] T034 [US2] Implement mark-as-read on click-through — when user clicks notification to view project, call markRead endpoint, update local state
- [ ] T035 [US2] Implement "Mark all as read" — calls markAllRead endpoint, resets unread count, updates list styling
- [ ] T036 [US1] Add NotificationBadge to global navigation bar (visible for authenticated Funders)
- [ ] T037 [US1] Write component tests for NotificationBadge in apps/<web-app>/src/features/notifications/__tests__/NotificationBadge.spec.tsx
- [ ] T038 [P] [US1] Write component tests for NotificationList and NotificationItem in apps/<web-app>/src/features/notifications/__tests__/NotificationList.spec.tsx
- [ ] T039 [P] [US2] Write component tests for mark-as-read flows in apps/<web-app>/src/features/notifications/__tests__/MarkRead.spec.tsx

**Checkpoint**: US1+US2 complete — Notification Center works, badge counts, read/unread tracking

---

## Phase 4: US3 + US4 — Email Notifications + Opt-out (Priority: P2)

**Goal**: Email notifications sent for the same events as in-app when user has email enabled. Users can toggle email on/off from account settings. Disabling email doesn't affect in-app.

**Independent Test**: Enable email → bookmarked project changes → verify email received. Disable email → another change → verify no email → verify in-app notification still appears.

### Backend — Email Service + Settings

- [ ] T040 [US3] Create NotificationEmailService in apps/<backend-app>/src/modules/notifications/notification-email.service.ts — sends email for a notification, uses existing email infrastructure, retries 3 times with exponential backoff on failure
- [ ] T041 [US3] Integrate email sending into NotificationGeneratorService — after creating in-app notification, check user's is_email_notifications_enabled flag, if true call NotificationEmailService
- [ ] T042 [P] [US4] Create UpdateNotificationPreferencesDto in apps/<backend-app>/src/modules/notifications/dto/update-preferences.dto.ts — validates isEmailNotificationsEnabled (boolean)
- [ ] T043 [US4] Implement GET /v1/notifications/settings in NotificationsController — returns current email preference
- [ ] T044 [US4] Implement PATCH /v1/notifications/settings in NotificationsController — updates email preference
- [ ] T045 [US3] Write unit tests for NotificationEmailService in apps/<backend-app>/src/modules/notifications/__tests__/notification-email.spec.ts — send, retry, failure logging
- [ ] T046 [P] [US4] Write unit tests for notification settings in apps/<backend-app>/src/modules/notifications/__tests__/notification-settings.spec.ts — get, update, email disabled stops emails

### Frontend — Settings

- [ ] T047 [US4] Create useNotificationSettings hook in apps/<web-app>/src/features/notifications/hooks/useNotificationSettings.ts — fetch and update email preference
- [ ] T048 [US4] Create NotificationSettingsPanel component in apps/<web-app>/src/features/notifications/ui/components/NotificationSettingsPanel.tsx — toggle for email notifications with label and save
- [ ] T049 [US4] Add NotificationSettingsPanel to account settings page — visible in the settings area alongside preferences
- [ ] T050 [US4] Write component tests for NotificationSettingsPanel in apps/<web-app>/src/features/notifications/__tests__/NotificationSettings.spec.tsx

**Checkpoint**: US3+US4 complete — emails sent when enabled, toggle works, in-app unaffected

---

## Phase 5: Polish & Cross-Cutting Concerns

- [ ] T051 [P] Add responsive layout for Notification Center — mobile 375px, tablet 768px, desktop 1440px
- [ ] T052 [P] Handle deleted project notifications — when user clicks a notification for a deleted project, show "Project no longer available" message instead of 404
- [ ] T053 [P] Write E2E test for notification center in tests/e2e/notifications/notification-center.spec.ts — view notifications, pagination, click-through
- [ ] T054 [P] Write E2E test for mark-as-read in tests/e2e/notifications/mark-read.spec.ts — mark one, mark all, badge update
- [ ] T055 [P] Write E2E test for email settings in tests/e2e/notifications/email-settings.spec.ts — toggle on/off
- [ ] T056 [P] Write E2E test for notification generation in tests/e2e/notifications/notification-generation.spec.ts — bookmark project, change project, verify notification appears
- [ ] T057 Run quickstart.md validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies
- **Phase 2 (Foundation)**: Depends on Phase 1
- **Phase 3 (US1+US2)**: Depends on Phase 2 — MVP notification center
- **Phase 4 (US3+US4)**: Depends on Phase 3 (extends with email)
- **Phase 5 (Polish)**: Depends on all stories

### User Story Dependencies

```
Phase 1 (Setup)
    │
Phase 2 (Foundation: DB + API client)
    │
Phase 3 (US1+US2: Center + Read Tracking) ─── MVP
    │
Phase 4 (US3+US4: Email + Opt-out)
    │
Phase 5 (Polish + E2E)
```

### Parallel Opportunities

- **Phase 1**: T001-T003 types in parallel
- **Phase 2**: T005-T007 migrations in parallel
- **Phase 3**: Backend generator (T017-T018) and frontend (T026-T039) can overlap after CRUD services done
- **Phase 3**: T023+T024 service tests in parallel, T037+T038+T039 component tests in parallel
- **Phase 5**: All E2E tests (T053-T056) in parallel

---

## Implementation Strategy

### MVP First (Phase 1 + 2 + 3)

1. Phase 1: Shared types
2. Phase 2: DB schema + API client
3. Phase 3: Notification Center + generation + badge + read tracking
4. **STOP and VALIDATE**: Funders see notifications for bookmark changes, badge works, mark-as-read works
5. Deploy/demo

### Incremental Delivery

1. **Phase 1+2+3** → In-app notifications work (MVP)
2. **Phase 4** → Email notifications + opt-out → Deploy
3. **Phase 5** → E2E + polish → Final release

---

## Notes

- US1+US2 grouped — center without read tracking is a noisy list
- US3+US4 grouped — email without opt-out violates user trust
- Background generator runs every 2-3 min (NestJS @Cron or @Interval)
- 10-minute batching window for rapid changes on same project
- 60-second polling for unread badge count
- Email failures retried 3x — never block in-app notification creation
- Notifications survive project deletion (SET NULL on FK)
