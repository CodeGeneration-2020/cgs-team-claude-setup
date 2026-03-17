# Research: Bookmarks

**Feature**: 004-bookmarks | **Date**: 2026-03-17

## R1: Bookmark Toggle UX Pattern

**Decision**: Optimistic UI with server reconciliation. Toggle the icon immediately on click, send the request in the background, revert on failure.

**Rationale**: SC-001 requires < 1 second perceived response. Optimistic updates give instant feedback. TanStack Query's `useMutation` with `onMutate`/`onError` rollback handles this pattern natively.

**Alternatives Considered**:
- **Wait for server response**: Adds visible delay (200-500ms). Poor UX for a simple toggle.
- **Local-only with periodic sync**: Complex, can lose data. Over-engineered.

---

## R2: Bookmark Storage

**Decision**: Simple `bookmarks` join table with unique constraint on (user_id, project_id). Toggle = INSERT on bookmark, DELETE on unbookmark.

**Rationale**: Bookmarks are a many-to-many relationship between users and projects. A join table is the simplest model. The unique constraint enforces idempotency (FR-009).

**Alternatives Considered**:
- **Array column on user**: Can't query "who bookmarked this project" or enforce referential integrity.
- **Soft-delete toggle (is_active flag)**: Adds complexity for no benefit — a deleted bookmark has no value to preserve.

---

## R3: Update Tracking Strategy

**Decision**: Derive bookmark updates from the existing `project_audit_log` table (from 003-project-management). When a Funder views their bookmarks, query audit log entries for their bookmarked project IDs, filtered to significant change types (EDITED with critical fields, status changes), created after their last visit or after the bookmark was created.

**Rationale**: The project audit log already captures all project changes with field-level detail. Building a separate event system would duplicate data. Deriving updates from audit logs is simple, consistent, and requires no additional write-path changes.

**Alternatives Considered**:
- **Separate bookmark_updates table**: Requires writing to two tables on every project change. More moving parts, more places for bugs.
- **Event-driven system (pub/sub)**: Scalable but over-engineered for the current user base. Can migrate later if needed.
- **Polling from frontend**: Bad for performance at scale. Server-side derivation is better.

**Implementation Notes**:
- Query: `SELECT * FROM project_audit_log WHERE project_id IN (user's bookmarked project IDs) AND action IN ('EDITED', 'APPROVED', 'REJECTED') AND created_at > bookmark.created_at ORDER BY created_at DESC`
- For EDITED actions, filter `changed_fields` to only show significant changes (description, funding, status, stakeholders).
- Add a `last_seen_at` timestamp on the bookmark record so we can show "new" indicators for unseen updates.

---

## R4: Stale Bookmark Handling

**Decision**: When a bookmarked project is deleted (hard delete per 003-project-management), the bookmark record becomes an orphan (project_id FK is null or dangling). Handle this gracefully by LEFT JOINing projects and marking missing projects as "No longer available."

**Rationale**: FR-013 requires stale bookmarks to be marked, not silently removed. LEFT JOIN lets us detect missing projects without a separate cleanup job.

**Alternatives Considered**:
- **CASCADE DELETE on FK**: Would silently remove bookmarks when a project is deleted. Violates FR-013.
- **Soft-delete projects instead**: Already decided against in 003-project-management (hard delete with audit snapshot).
- **ON DELETE SET NULL**: Keeps the bookmark row with null project_id. Works but loses the project_id for any future reference. Better to use LEFT JOIN and show "unavailable."

**Implementation Notes**:
- Prisma: Set `onDelete: SetNull` or handle at application level.
- In the bookmarks list response, include a `isAvailable: boolean` field. Frontend renders "No longer available" for false.

---

## R5: BookmarkButton as Shared Component

**Decision**: Create BookmarkButton in `/packages/ui-components` since it appears on ProjectCard across homepage search results, trending section, and project detail pages.

**Rationale**: Constitution Principle IX (Shared-Before-Custom). The button must be consistent everywhere ProjectCard appears. Making it a shared component ensures visual consistency and deduplication.

**Implementation Notes**:
- Component accepts: `isBookmarked: boolean`, `onToggle: () => void`, `isLoading?: boolean`.
- Does not contain business logic — the hook (`useBookmarkToggle`) in the bookmarks feature module handles the API call.
- Renders a heart/bookmark icon with filled/outline states.
