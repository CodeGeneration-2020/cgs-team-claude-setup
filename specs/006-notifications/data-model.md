# Data Model: Notifications

**Feature**: 006-notifications | **Date**: 2026-03-17

## Entities

### Notification (new)

An in-app notification for a user about a project event.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| user_id | String | FK to User (the Funder receiving the notification) | Required |
| project_id | String | FK to Project (nullable — project may be deleted) | SET NULL on project delete |
| event_type | Enum (NotificationEventType) | Category of the notification | Required |
| title | String | Short notification title | Required, max 200 chars |
| message | String | Notification body text | Required, max 1000 chars |
| is_read | Boolean | Whether the user has seen/acknowledged it | Default: false |
| change_count | Integer | Number of batched changes (for rapid updates) | Default: 1 |
| source_audit_id | String | FK to project_audit_log entry that triggered it | Nullable |
| created_at | DateTime | When the notification was created | Auto-generated |
| read_at | DateTime | When the notification was marked as read | Nullable |

**Relationships:**
- Notification belongs to User (many-to-one)
- Notification belongs to Project (many-to-one, nullable)

**Constraints:**
- Notifications are append-only (never deleted through the application, only marked as read).
- `project_id` uses SET NULL so notifications survive project deletion.

**Indexes:**
- `idx_notifications_user_id_created_at` on `(user_id, created_at DESC)` for paginated list
- `idx_notifications_user_id_is_read` on `(user_id, is_read)` for unread count
- `idx_notifications_user_id_project_id_created_at` on `(user_id, project_id, created_at)` for batching check

---

### User / UserPreferences (extended)

Add email notification preference field.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| is_email_notifications_enabled | Boolean | Whether the user wants email notifications | Default: true |

This field is added to the existing User or UserPreferences entity (from 005-onboarding).

---

### NotificationProcessorState (new, internal)

Tracks the background job's processing position to avoid re-processing events.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String | Fixed key (e.g., "notification_processor") | Primary key |
| last_processed_at | DateTime | Timestamp of the last processed audit log entry | Required |
| updated_at | DateTime | When this state was last updated | Auto-generated |

**Notes:**
- Single-row table. Only one processor state exists at a time.
- Used by NotificationGeneratorService to know where to start scanning audit logs.

---

## Enum Definitions

### NotificationEventType

| Value | Description |
|-------|-------------|
| BOOKMARK_PROJECT_UPDATED | A bookmarked project's description/funding/stakeholders changed |
| BOOKMARK_PROJECT_STATUS_CHANGED | A bookmarked project's status changed |
| NEW_MATCHING_PROJECT | A new project matches the user's onboarding preferences |
| PROJECT_NO_LONGER_AVAILABLE | A bookmarked project was deleted or archived |
