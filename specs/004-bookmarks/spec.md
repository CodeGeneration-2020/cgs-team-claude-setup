# Feature Specification: Bookmarks

**Feature Branch**: `004-bookmarks`
**Created**: 2026-03-17
**Status**: Draft
**Input**: User description: "Bookmarks — funders can bookmark projects, view bookmarked projects, track updates on bookmarked projects, and manage their bookmark collection"

## User Scenarios & Testing *(mandatory)*

### US1 — Bookmark a Project (Priority: P1)

As a Funder, I want to bookmark projects so that I can save them for later review.

**Why this priority**: Bookmarking is the core action — without it, the rest of the feature doesn't exist. This is the most fundamental funder engagement action.

**Independent Test**: Log in as Funder → view a project → click bookmark → verify bookmark indicator appears → navigate away and return → verify bookmark persists.

**Acceptance Scenarios**:

1. **Given** a Funder is viewing a project (in search results or on the detail page), **When** they click the bookmark icon/button, **Then** the project is bookmarked and the icon changes to indicate it is saved.
2. **Given** a Funder has bookmarked a project, **When** they view the same project again, **Then** the bookmark icon shows it is already bookmarked.
3. **Given** a Funder has bookmarked a project, **When** they click the bookmark icon again, **Then** the bookmark is removed and the icon reverts to the un-bookmarked state.
4. **Given** a Funder bookmarks or un-bookmarks a project, **When** the action completes, **Then** the browsing experience is not interrupted — the user stays on the same page, same scroll position.
5. **Given** a Funder bookmarks a project, **When** they log out and log back in, **Then** the bookmark is still saved.
6. **Given** a Visitor (not logged in) views a project, **When** they try to bookmark it, **Then** they are prompted to log in or create an account.

---

### US2 — View My Bookmarks (Priority: P1)

As a Funder, I want to view all my bookmarked projects in one place so that I can easily review projects I'm interested in.

**Why this priority**: Without a bookmarks list, bookmarks have no discoverability. Users need a dedicated place to see everything they've saved.

**Independent Test**: Log in as Funder → bookmark several projects → navigate to "My Bookmarks" → verify all bookmarked projects appear with key info.

**Acceptance Scenarios**:

1. **Given** a Funder is logged in, **When** they navigate to "My Bookmarks" (via account menu or navigation), **Then** they see a list of all projects they have bookmarked.
2. **Given** the bookmarks list is displayed, **When** the Funder looks at each entry, **Then** they can see: project title, organization name, thematic area, location, project status, and short description (same info as search result cards).
3. **Given** a Funder sees a bookmarked project in the list, **When** they click on it, **Then** they are navigated to the project's detail page.
4. **Given** a Funder has no bookmarked projects, **When** they open "My Bookmarks," **Then** they see an empty state message: "You haven't bookmarked any projects yet" with a link to browse/search projects.
5. **Given** a Funder views the bookmarks list, **When** they click the bookmark icon on a listed project, **Then** the project is removed from their bookmarks and disappears from the list.

---

### US3 — Track Updates on Bookmarked Projects (Priority: P2)

As a Funder, I want to see updates about my bookmarked projects so that I stay informed about changes without constantly checking each project.

**Why this priority**: Update tracking adds ongoing value to bookmarks, turning them from a static list into a monitoring tool. However, basic bookmark/unbookmark works without this.

**Independent Test**: Bookmark a project → project owner updates the description → navigate to My Bookmarks → verify an update indicator is shown for that project.

**Acceptance Scenarios**:

1. **Given** a Funder has bookmarked a project, **When** a significant change occurs on that project (description update, status change, stakeholder change, funding stage change), **Then** the Funder can see an update indicator in their "My Bookmarks" area.
2. **Given** updates exist for bookmarked projects, **When** the Funder views "My Bookmarks," **Then** each update clearly indicates which project it relates to and what changed.
3. **Given** a Funder sees an update for a bookmarked project, **When** they click on the update, **Then** they are navigated directly to the updated project's detail page.
4. **Given** a Funder removes a bookmark from a project, **When** they view their updates, **Then** they no longer see updates for that project.
5. **Given** a bookmarked project is deleted or archived by the owner, **When** the Funder views their bookmarks, **Then** the project is marked as "No longer available" and the bookmark can be removed.

---

### Edge Cases

- What happens when a Funder bookmarks a project that is later rejected or archived? The bookmark persists but the project is marked as "No longer available" in the bookmarks list. The Funder can remove the stale bookmark.
- What happens when a Funder has hundreds of bookmarks? The bookmarks list should be paginated (e.g., 20 per page).
- What happens when a Funder bookmarks the same project twice (e.g., race condition from two browser tabs)? The system should handle idempotently — only one bookmark record is created.
- What happens when the bookmark toggle fails due to network error? The UI should revert the optimistic update and show a brief error message: "Could not save bookmark. Please try again."
- What happens when a project's status changes to Inactive while a Funder is viewing it? The bookmark icon should still work, but on the next visit to bookmarks, the project shows its updated status.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow authenticated Funders to bookmark any active/approved project.
- **FR-002**: System MUST allow Funders to remove a bookmark from a project (toggle behavior).
- **FR-003**: Bookmarking or removing a bookmark MUST NOT interrupt the user's current browsing (no page reload, no scroll reset).
- **FR-004**: Bookmarks MUST persist across sessions — saved bookmarks are retained when the user logs out and back in.
- **FR-005**: System MUST provide a "My Bookmarks" page accessible from the user's account area or main navigation.
- **FR-006**: The bookmarks list MUST display each project with: title, organization name, thematic area, location, status, short description — using the same card format as search results.
- **FR-007**: System MUST display an empty state message when a Funder has no bookmarks, with a link to browse projects.
- **FR-008**: System MUST show a bookmark indicator on project cards (in search results and trending section) and on the project detail page, reflecting whether the current user has bookmarked that project.
- **FR-009**: System MUST handle bookmark operations idempotently — bookmarking an already-bookmarked project is a no-op, not an error.
- **FR-010**: System MUST track significant changes on bookmarked projects and make them visible to the Funder in the "My Bookmarks" area. Significant changes include: description update, status change, stakeholder change, funding stage change.
- **FR-011**: Each update MUST clearly indicate which project it relates to and what changed.
- **FR-012**: When a Funder removes a bookmark, they MUST stop seeing updates for that project.
- **FR-013**: When a bookmarked project is deleted or archived, the system MUST mark it as "No longer available" in the bookmarks list rather than silently removing it.
- **FR-014**: The bookmarks list MUST be paginated when the Funder has more than 20 bookmarks.
- **FR-015**: Only authenticated users with the Funder role (or Super Admin) MUST be able to bookmark projects. Visitors MUST be prompted to log in.
- **FR-016**: Bookmark operations MUST use optimistic UI updates — the icon toggles immediately, reverting on failure.

### Key Entities

- **Bookmark**: A saved reference to a project by a Funder. Key attributes: id, user id (Funder), project id, created at. Unique constraint on (user_id, project_id).
- **BookmarkUpdate**: A tracked change on a bookmarked project. Key attributes: id, project id, change type (description_updated, status_changed, stakeholder_changed, funding_changed), change summary, created at. Updates are visible to all Funders who have bookmarked the project.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Funders can bookmark or un-bookmark a project in under 1 second (perceived response time with optimistic UI).
- **SC-002**: The "My Bookmarks" page loads within 2 seconds showing all bookmarked projects.
- **SC-003**: 100% of bookmark operations persist correctly across sessions — no lost bookmarks after logout/login.
- **SC-004**: Updates on bookmarked projects appear in the Funder's bookmarks area within 5 minutes of the change occurring.
- **SC-005**: Stale bookmarks (deleted/archived projects) are clearly marked, not silently hidden.
- **SC-006**: The bookmark feature works correctly on desktop, tablet, and mobile.

## Scope

### In Scope
- Bookmark/un-bookmark toggle on project cards and detail pages
- "My Bookmarks" page with project card list and pagination
- Bookmark indicator on project cards throughout the platform
- Bookmark persistence across sessions
- Update tracking for bookmarked projects (in-app, within the bookmarks area)
- Empty state for no bookmarks
- Stale bookmark handling (deleted/archived projects)
- Optimistic UI for bookmark toggle
- Login prompt for unauthenticated bookmark attempts

### Out of Scope
- Email notifications for bookmark updates (separate Notifications feature — US-25)
- Push notifications (deferred)
- Bookmark folders/categories/tags (deferred — can organize bookmarks later)
- Bookmark sharing with other users (deferred)
- Bookmark export (deferred)
- Org Admin bookmarking (only Funders and Super Admins can bookmark)

## Assumptions

- Registration & Roles (002-registration-roles) is implemented — Funder role exists with RBAC.
- Homepage search (001-homepage) exists — project cards in search results can display the bookmark icon.
- Project Management (003-project-management) exists — projects have statuses, descriptions, and audit events that can trigger bookmark updates.
- The ProjectCard shared component (from 001-homepage) can accept an optional bookmark toggle prop.
- Bookmark updates are generated when project audit log events occur (from 003-project-management audit logging). No separate event system is needed — updates can be derived from existing audit logs.
