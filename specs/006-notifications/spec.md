# Feature Specification: Notifications

**Feature Branch**: `006-notifications`
**Created**: 2026-03-17
**Status**: Draft
**Input**: User description: "Notifications — in-app notification center and email notifications for bookmark updates, project status changes, and profile-matching new projects"

## User Scenarios & Testing *(mandatory)*

### US1 — In-App Notification Center (Priority: P1)

As a Funder, I want a dedicated Notifications page on the platform so that I can see all changes related to my bookmarked projects and relevant new projects in one place.

**Why this priority**: The notification center is the core delivery mechanism. Without it, users have no way to see platform activity relevant to them. Email is secondary — in-app is the primary channel.

**Independent Test**: Log in as Funder with bookmarked projects → a bookmarked project gets updated → navigate to Notification Center → verify notification appears with project name and change description.

**Acceptance Scenarios**:

1. **Given** a Funder is logged in, **When** they click the notifications icon in the navigation, **Then** they are taken to the Notification Center page showing all their notifications.
2. **Given** a bookmarked project's status changes (e.g., Approved → Archived), **When** the Funder opens the Notification Center, **Then** they see a notification: "[Project Name] status changed to Archived."
3. **Given** a bookmarked project's description or funding is updated, **When** the Funder opens the Notification Center, **Then** they see a notification describing the change.
4. **Given** a new project is added that strongly matches the Funder's onboarding profile, **When** the Funder opens the Notification Center, **Then** they see a notification: "New project matching your interests: [Project Name]."
5. **Given** multiple notifications exist, **When** the Funder views the Notification Center, **Then** notifications are ordered by most recent first.
6. **Given** a notification is displayed, **When** the Funder reads it, **Then** they can click on it to navigate directly to the related project's detail page.
7. **Given** the Funder has unread notifications, **When** they view the navigation bar, **Then** a badge/counter shows the number of unread notifications.

---

### US2 — Mark Notifications as Read (Priority: P1)

As a Funder, I want to mark notifications as read so that I can track which updates I've already seen.

**Why this priority**: Without read/unread tracking, the notification center becomes a noisy, unusable list. This is essential for the center to function properly.

**Independent Test**: View Notification Center → see unread notifications → mark one as read → verify it visually changes → verify unread counter decreases.

**Acceptance Scenarios**:

1. **Given** a Funder has unread notifications, **When** they view the Notification Center, **Then** unread notifications are visually distinct from read ones (e.g., bold text, highlight, or dot indicator).
2. **Given** a Funder clicks on a notification to view the related project, **When** they navigate to the project, **Then** the notification is automatically marked as read.
3. **Given** a Funder wants to mark a notification as read without navigating, **When** they use a "Mark as read" action on the notification, **Then** it changes to read state.
4. **Given** the Funder marks a notification as read, **When** the unread counter in the navigation updates, **Then** it decreases by one.
5. **Given** the Funder has many unread notifications, **When** they click "Mark all as read," **Then** all notifications become read and the counter resets to zero.

---

### US3 — Email Notifications (Priority: P2)

As a Funder, I want to receive email notifications about changes in my bookmarked projects so that I stay informed without constantly checking the platform.

**Why this priority**: Email extends reach beyond active platform usage. However, the in-app center must work first — email mirrors the same events.

**Independent Test**: Bookmark a project → project owner updates it → check email inbox → verify email received with project name, change description, and link to the project.

**Acceptance Scenarios**:

1. **Given** a Funder has email notifications enabled (default: on), **When** a bookmarked project changes, **Then** they receive an email notification with the project name, change description, and a link to view the project.
2. **Given** a Funder has email notifications enabled, **When** a new project matching their profile is added, **Then** they receive an email about the new project.
3. **Given** an email notification is sent, **When** the Funder clicks the link in the email, **Then** they are taken directly to the project's detail page on the platform.
4. **Given** a Funder has email notifications disabled, **When** a bookmarked project changes, **Then** no email is sent, but the in-app notification still appears.

---

### US4 — Opt Out of Email Notifications (Priority: P2)

As a Funder, I want to opt out of email notifications so that I only receive in-app notifications.

**Why this priority**: User control over email is essential for trust and compliance. Tightly coupled with US3 — if email exists, opt-out must exist.

**Independent Test**: Log in → account settings → Notification Settings → disable email toggle → verify no more emails received → verify in-app notifications still work.

**Acceptance Scenarios**:

1. **Given** a Funder opens their account settings, **When** they navigate to Notification Settings, **Then** they see a toggle for Email Notifications (default: enabled).
2. **Given** a Funder disables email notifications, **When** they save the setting, **Then** they stop receiving email notifications.
3. **Given** email notifications are disabled, **When** a bookmarked project changes, **Then** the in-app notification still appears in the Notification Center.
4. **Given** a Funder has disabled email notifications, **When** they return to Notification Settings later, **Then** the toggle reflects the current state (disabled).
5. **Given** a Funder has disabled email notifications, **When** they re-enable them, **Then** they start receiving email notifications again for future events.

---

### Edge Cases

- What happens when a Funder has hundreds of notifications? The Notification Center should be paginated (20 per page).
- What happens when a notification references a project that has been deleted? The notification remains but clicking it shows a "Project no longer available" message.
- What happens when the email service is temporarily unavailable? Email failures should be retried. In-app notifications are unaffected.
- What happens when a Funder removes a bookmark? They stop receiving new notifications for that project, but existing notifications remain visible in the center.
- What happens when multiple changes occur on the same project in a short time? The system should batch related changes into a single notification rather than flooding the user (e.g., "3 updates on [Project Name]").
- What happens when a Visitor (not logged in) triggers a project change? The notification is generated from the event, not from who triggered it — all Funders who bookmarked the project are notified.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide an in-app Notification Center page accessible to authenticated Funders.
- **FR-002**: System MUST generate in-app notifications for the following events:
  - Bookmarked project status changes
  - Bookmarked project description/funding/stakeholder updates
  - New projects strongly matching the Funder's onboarding profile
- **FR-003**: Each notification MUST include: related project name, description of what changed, timestamp, and a link to the project.
- **FR-004**: Notifications MUST be ordered by most recent first.
- **FR-005**: System MUST display an unread notification count badge in the navigation bar.
- **FR-006**: System MUST visually distinguish unread notifications from read notifications.
- **FR-007**: System MUST mark a notification as read when the user clicks through to the related project.
- **FR-008**: System MUST provide a "Mark as read" action on individual notifications.
- **FR-009**: System MUST provide a "Mark all as read" action in the Notification Center.
- **FR-010**: System MUST send email notifications for the same events as in-app notifications when the user has email enabled.
- **FR-011**: Email notifications MUST include: project name, change description, and a direct link to the project on the platform.
- **FR-012**: Email notifications MUST be enabled by default for new accounts.
- **FR-013**: System MUST provide a Notification Settings page in account settings with a toggle to enable/disable email notifications.
- **FR-014**: Disabling email notifications MUST NOT affect in-app notifications.
- **FR-015**: System MUST save the user's email notification preference in their profile.
- **FR-016**: When a Funder removes a bookmark, the system MUST stop generating new notifications for that project. Existing notifications MUST remain visible.
- **FR-017**: Notification Center MUST be paginated (20 per page).
- **FR-018**: Notification events MUST be logged for system auditing.
- **FR-019**: System SHOULD batch multiple rapid changes on the same project into a single notification rather than sending separate notifications for each change.

### Key Entities

- **Notification**: An in-app notification for a user. Key attributes: id, user id, project id, event type, title, message, is read flag, created at. One user can have many notifications.
- **Notification Preference**: User's email notification setting. Key attributes: user id, is email enabled (default: true). One-to-one with User. Can be added as a field on the existing User or UserPreferences entity.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In-app notifications appear in the Notification Center within 5 minutes of the triggering event.
- **SC-002**: Email notifications are delivered within 10 minutes of the triggering event (when email is enabled).
- **SC-003**: The unread badge count is accurate within 1 minute of a new notification being created.
- **SC-004**: 100% of bookmark-related project changes generate corresponding notifications for all Funders who bookmarked the project.
- **SC-005**: Users can disable email notifications in under 30 seconds from account settings.
- **SC-006**: The Notification Center page loads within 2 seconds.
- **SC-007**: Disabling email has zero effect on in-app notification delivery.

## Scope

### In Scope
- In-app Notification Center page with paginated notification list
- Unread count badge in navigation
- Read/unread tracking with mark-as-read and mark-all-as-read
- Notification generation for: bookmark project updates, bookmark project status changes, new profile-matching projects
- Email notification delivery for the same events
- Email notification opt-out toggle in account settings
- Notification batching for rapid changes on the same project
- Click-through from notification to project detail
- Notification logging for auditing

### Out of Scope
- Push notifications (mobile/browser) — deferred
- Notification preferences per event type (e.g., only email for status changes) — deferred, single on/off toggle for now
- Org Admin notifications (e.g., project approved/rejected) — the rejection reason is shown in My Projects (003-project-management), not via the notification system
- Real-time notifications (WebSocket) — notifications are loaded on page visit and poll/refresh, not pushed in real-time
- Notification digest emails (daily/weekly summary) — deferred
- SMS notifications — deferred

## Assumptions

- Bookmarks feature (004-bookmarks) is implemented — notifications are triggered by changes to bookmarked projects.
- Onboarding (005-onboarding) is implemented — "new profile-matching projects" notifications use the user's preferences data.
- Project Management (003-project-management) is implemented — project audit log events trigger bookmark-related notifications.
- Email sending infrastructure exists (from 002-registration-roles password recovery).
- The notification generation process runs periodically or is triggered by project changes — it does not require real-time event streaming.
- A single email toggle (on/off) is sufficient for the initial release. Per-event-type preferences can be added later.
