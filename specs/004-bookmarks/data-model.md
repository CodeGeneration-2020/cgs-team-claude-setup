# Data Model: Bookmarks

**Feature**: 004-bookmarks | **Date**: 2026-03-17

## Entities

### Bookmark (new)

A saved reference to a project by a Funder.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| user_id | String | FK to User (the Funder) | Required |
| project_id | String | FK to Project (nullable — project may be deleted) | Required at creation, SET NULL on project delete |
| last_seen_at | DateTime | When the Funder last viewed updates for this bookmark | Default: created_at |
| created_at | DateTime | When the bookmark was created | Auto-generated |

**Relationships:**
- Bookmark belongs to User (many-to-one)
- Bookmark belongs to Project (many-to-one, nullable after project deletion)

**Constraints:**
- Unique constraint on `(user_id, project_id)` — a user can bookmark a project at most once.
- `project_id` uses ON DELETE SET NULL so bookmarks survive project deletion.
- Only users with FUNDER or SUPER_ADMIN role can create bookmarks.

**Indexes:**
- `idx_bookmarks_user_id` on `user_id` for "My Bookmarks" queries
- `idx_bookmarks_project_id` on `project_id` for "who bookmarked this" queries
- `idx_bookmarks_user_project` on `(user_id, project_id)` unique for toggle lookups

---

## Derived Data (no new table)

### Bookmark Updates

Updates are **derived from the existing `project_audit_log` table** (from 003-project-management). No separate table is needed.

**Query pattern:**
```
For a given user's bookmarks:
1. Get all project_ids from user's bookmarks
2. Query project_audit_log WHERE:
   - project_id IN (bookmarked project IDs)
   - action IN ('EDITED', 'APPROVED', 'REJECTED')
   - created_at > bookmark.last_seen_at (for "new" indicator)
3. For EDITED actions, filter changed_fields to significant changes:
   - description, funding_requested, status, stakeholder_sector, thematic_areas
4. Return as BookmarkUpdate objects
```

**BookmarkUpdate shape (response only, not stored):**

| Field | Type | Description |
|-------|------|-------------|
| projectId | String | Which project changed |
| projectTitle | String | Project title (for display) |
| changeType | String | What kind of change (description_updated, status_changed, funding_changed, etc.) |
| changeSummary | String | Human-readable summary of what changed |
| occurredAt | DateTime | When the change happened |
| isNew | Boolean | Whether this update is newer than last_seen_at |

---

## No Enum Definitions

This feature uses existing enums (ProjectAction from 003-project-management for audit log queries).
