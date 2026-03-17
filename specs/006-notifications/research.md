# Research: Notifications

**Feature**: 006-notifications | **Date**: 2026-03-17

## R1: Notification Generation Strategy

**Decision**: Use a background job (NestJS scheduled task or cron) that periodically scans the project_audit_log for new events since last run, generates notifications for affected users (Funders who bookmarked the changed projects), and stores them in the notifications table.

**Rationale**: The spec says "no real-time WebSocket" and "within 5 minutes of event." A polling-based background job is the simplest approach. It reads from the existing audit log (003-project-management) and bookmark records (004-bookmarks) — no new event system needed. Runs every 2-3 minutes.

**Alternatives Considered**:
- **Inline generation (on project save)**: Simpler but couples notification logic to project write path. Slows down project saves if many bookmarkers exist. Background is cleaner.
- **Event-driven (pub/sub)**: More scalable but over-engineered for the current volume. Can migrate later.
- **Real-time WebSocket**: Explicitly out of scope per spec.

**Implementation Notes**:
- Track `last_processed_audit_id` or `last_processed_at` to avoid re-processing events.
- For each audit event: find all Funders who bookmarked that project → create one notification per Funder.
- For new project matching: run recommendation matching for all Funders with preferences, limit to high-confidence matches.

---

## R2: Notification Batching

**Decision**: Batch multiple rapid changes on the same project for the same user into a single notification. If multiple audit events for the same project occur within a 10-minute window, combine them into one notification: "3 updates on [Project Name]."

**Rationale**: FR-019 requires batching to avoid flooding users. A time-window approach is simple and effective.

**Implementation Notes**:
- Before creating a notification, check if a notification for the same user + project exists within the last 10 minutes and is still unread.
- If yes: update the existing notification message and increment the change count.
- If no: create a new notification.

---

## R3: Email Delivery

**Decision**: Send emails using the existing email service (from 002-registration-roles password recovery). Email is sent alongside notification creation when the user has email enabled. Email failures are retried (3 attempts with exponential backoff) and logged but do not block in-app notification creation.

**Rationale**: Reuses existing email infrastructure (Constitution Principle IX). Email is secondary — in-app is the source of truth. Retry handles transient failures.

**Alternatives Considered**:
- **Queue-based email**: More robust but requires a message queue. Deferred.
- **Separate email digest (daily/weekly)**: Explicitly out of scope.

---

## R4: Notification Preference Storage

**Decision**: Add an `is_email_notifications_enabled` boolean field on the existing User model (or UserPreferences from 005-onboarding). Default: true.

**Rationale**: A single toggle is all the spec requires (FR-013). Adding it to the existing user/preferences entity avoids a new table for one field.

**Alternatives Considered**:
- **Separate notification_preferences table**: Over-engineered for one boolean. Can migrate later if per-event-type preferences are needed.
- **JSON preferences column**: Flexible but unnecessary for a single toggle.

---

## R5: Unread Count Delivery

**Decision**: The unread count is fetched via a lightweight API endpoint (`GET /v1/notifications/unread-count`) that the frontend polls every 60 seconds. The count is also returned with the notifications list response.

**Rationale**: SC-003 requires "accurate within 1 minute." Polling every 60 seconds meets this. A dedicated count endpoint avoids loading the full notification list just to get the badge number.

**Alternatives Considered**:
- **WebSocket push**: Out of scope.
- **Include in every API response header**: Adds coupling between notification system and all other endpoints.
