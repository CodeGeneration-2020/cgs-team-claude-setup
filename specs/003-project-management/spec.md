# Feature Specification: Project Management

**Feature Branch**: `003-project-management`
**Created**: 2026-03-17
**Status**: Draft
**Input**: User description: "Project management — Org Admins create, edit, publish, delete projects with structured forms, predefined taxonomy tags, Super Admin approval workflow, and audit logging"

## User Scenarios & Testing *(mandatory)*

### US1 — Create a Project (Priority: P1)

As an Org Admin, I want to create a project using a structured form so that I can publish my funding needs on the platform.

**Why this priority**: Project creation is the core action for organizations — without it, the platform has no content for funders to discover. This is the single most critical project management function.

**Independent Test**: Log in as Org Admin → navigate to "Create Project" → fill all required fields → submit → verify project is saved with status "Pending Approval" and appears in "My Projects."

**Acceptance Scenarios**:

1. **Given** an Org Admin is logged in, **When** they navigate to the project section, **Then** they see a "Create New Project" option.
2. **Given** an Org Admin opens the project creation form, **When** the form loads, **Then** they see a structured form with predefined fields including: thematic area, geographic location, lifecycle status, requested funding, implementer type, title, description, and other project details.
3. **Given** an Org Admin fills in the form, **When** they select a thematic area, **Then** only predefined taxonomy values are available for selection (no free-text entry for taxonomy fields).
4. **Given** an Org Admin has filled all required fields, **When** they click "Save / Publish project," **Then** the system validates the form and saves the project with status "Pending Approval."
5. **Given** an Org Admin leaves a required field empty, **When** they attempt to submit, **Then** the system prevents submission and highlights the missing required fields.
6. **Given** a project is successfully created, **When** the Org Admin navigates to "My Projects," **Then** the new project appears in the list with status "Pending Approval."
7. **Given** a project is created, **When** the system processes the creation, **Then** an audit log entry is generated recording who created the project and when.

---

### US2 — View My Projects (Priority: P1)

As an Org Admin, I want to see all projects I created so that I can manage them easily.

**Why this priority**: Without a project list, Org Admins cannot manage, edit, or track the status of their submissions. This is essential alongside creation.

**Independent Test**: Log in as Org Admin → navigate to "My Projects" → verify project list with correct statuses and actions.

**Acceptance Scenarios**:

1. **Given** an Org Admin is logged in, **When** they navigate to "My Projects," **Then** they see a list of all projects they have created.
2. **Given** the project list is displayed, **When** the Org Admin looks at a project entry, **Then** they can see: project title, thematic area, location, funding requested, and project status.
3. **Given** projects have various statuses, **When** the list is displayed, **Then** status indicators include: Draft, Pending Approval, Approved, Rejected, and Archived.
4. **Given** an Org Admin sees a project in the list, **When** they click on it, **Then** they can: view full details, edit the project, or delete the project.

---

### US3 — Publish a Project (Priority: P1)

As an Org Admin, I want to publish a project so that it becomes visible to potential funders after admin approval.

**Why this priority**: Publishing triggers the approval workflow — it's the bridge between creation and public visibility. Without it, projects remain in draft forever.

**Independent Test**: Create a project → click "Publish" → confirm in modal → verify status changes to "Pending Approval" and project enters the admin review queue.

**Acceptance Scenarios**:

1. **Given** an Org Admin has completed the project form, **When** they click "Publish Project," **Then** a confirmation modal appears: "Publish this project?"
2. **Given** the confirmation modal is displayed, **When** the Org Admin clicks "Yes, publish," **Then** the project status becomes "Pending Approval."
3. **Given** the confirmation modal is displayed, **When** the Org Admin clicks "Cancel," **Then** no status change occurs and they return to the form.
4. **Given** a project is published, **When** the status is "Pending Approval," **Then** the project is added to the Super Admin review queue.
5. **Given** a project has status "Pending Approval," **When** a visitor searches the platform, **Then** the project is NOT visible in search results (only approved projects are public).
6. **Given** a project is published, **When** the Org Admin views "My Projects," **Then** they can see the status "Pending Approval" next to the project.

---

### US4 — Edit a Project (Priority: P2)

As an Org Admin, I want to edit an existing project so that I can update project information when things change.

**Why this priority**: Projects evolve — funding needs change, timelines shift, partners join. Editing is critical for keeping data current but can be delivered after the create/publish/approve flow works.

**Independent Test**: Log in as Org Admin → open "My Projects" → select a project → edit fields → save → verify changes are persisted.

**Acceptance Scenarios**:

1. **Given** an Org Admin opens "My Projects," **When** they select a project to edit, **Then** the Edit Project screen opens with the same structured fields used in creation, pre-filled with current values.
2. **Given** an Org Admin is editing a project, **When** they modify editable fields (description, funding information, timeline, tags, partners, sustainability info), **Then** the changes are reflected in the form.
3. **Given** an Org Admin has made changes, **When** they click "Save changes," **Then** the system validates the form (required fields still present) and updates the project.
4. **Given** an approved project is edited, **When** critical fields change (funding amount, thematic area, location), **Then** the system flags the update for re-review by noting it was modified after approval.
5. **Given** a project is edited, **When** the save completes, **Then** an audit log entry is created capturing: who edited, what fields changed, and timestamp.

---

### US5 — Delete a Project (Priority: P2)

As an Org Admin, I want to delete a project so that I can remove incorrect or duplicate submissions.

**Why this priority**: Important for data hygiene but less urgent than create/publish. Soft deletion or archival could serve as a temporary alternative.

**Independent Test**: Log in as Org Admin → open a project → click "Delete" → confirm → verify project is removed from all public views and search.

**Acceptance Scenarios**:

1. **Given** an Org Admin views a project they own, **When** they look at the available actions, **Then** they see a "Delete project" option.
2. **Given** an Org Admin clicks "Delete," **When** the action is triggered, **Then** a confirmation modal appears: "Are you sure you want to delete this project?"
3. **Given** the confirmation modal is shown, **When** the Org Admin clicks "Delete," **Then** the project is removed from the system.
4. **Given** the confirmation modal is shown, **When** the Org Admin clicks "Cancel," **Then** no deletion occurs.
5. **Given** a project is deleted, **When** visitors search the platform, **Then** the deleted project no longer appears in search results or dashboards.
6. **Given** a project is deleted, **When** the system processes the deletion, **Then** the event is recorded in audit logs.

---

### US6 — Approve or Reject a Project (Priority: P1)

As a Super Admin, I want to review and approve or reject submitted projects so that only verified projects are published on the platform.

**Why this priority**: The approval gate ensures data quality — without it, any content could appear publicly. This completes the publish flow started in US3.

**Independent Test**: Log in as Super Admin → open pending projects queue → review a project → approve it → verify it appears in public search. Reject another → verify rejection reason is sent to owner.

**Acceptance Scenarios**:

1. **Given** a Super Admin is logged in, **When** they navigate to the project review section, **Then** they see a list of projects with status "Pending Approval."
2. **Given** a Super Admin opens a pending project, **When** they review its details, **Then** they can see all project information submitted by the Org Admin.
3. **Given** a Super Admin is reviewing a project, **When** they click "Approve," **Then** the project status changes to "Approved" and it becomes visible to visitors in search results.
4. **Given** a Super Admin is reviewing a project, **When** they click "Reject," **Then** they are prompted to provide a rejection reason.
5. **Given** a Super Admin enters a rejection reason and confirms, **When** the rejection is processed, **Then** the project status changes to "Rejected" and the project owner is notified with the reason.
6. **Given** a project is approved or rejected, **When** the decision is processed, **Then** an audit log entry records: who made the decision, the decision (approved/rejected), the reason (if rejected), and the timestamp.

---

### US7 — Assign Predefined Tags to Project (Priority: P2)

As an Org Admin, I want to assign predefined classification values to a project so that it appears correctly in search results and dashboards.

**Why this priority**: Tags power the filtering and dashboard features. Without them, search filters return no results and dashboards have no data to aggregate. However, basic project creation works without rich tagging.

**Independent Test**: Create a project → select predefined tags from taxonomy dropdowns → save → verify tags appear on project card and detail page, and are usable in search filters.

**Acceptance Scenarios**:

1. **Given** an Org Admin is creating or editing a project, **When** they reach taxonomy fields (thematic area, stakeholder sector, etc.), **Then** they can only select from predefined values — not enter free text.
2. **Given** taxonomy fields allow multiple selections, **When** the Org Admin selects multiple values, **Then** all selected values are saved and associated with the project.
3. **Given** an Org Admin attempts to submit a value not in the predefined list, **When** validation runs, **Then** the invalid value is rejected.
4. **Given** a project has predefined tags assigned, **When** the project appears in search results or its detail page, **Then** the assigned tags are displayed as visual labels.
5. **Given** predefined tags are assigned to projects, **When** a visitor uses filters on the search page, **Then** the filter options reflect the predefined taxonomy values and correctly filter projects.

---

### Edge Cases

- What happens when an Org Admin tries to create a project but the predefined taxonomy list is empty? The system should display a message: "Project creation is temporarily unavailable" and log the configuration issue.
- What happens when a Super Admin approves a project that has since been deleted by the Org Admin? The system should handle the race condition — if the project no longer exists, show a message: "This project has been removed by the owner."
- What happens when an Org Admin edits a project that is currently being reviewed by a Super Admin? The edit should be allowed, but the Super Admin should see the latest version when they make their decision.
- What happens when a project is rejected — can it be resubmitted? Yes, the Org Admin can edit the rejected project and re-publish it, which sends it back to "Pending Approval."
- What happens when the Org Admin's account is converted to Funder while they have draft projects? Draft projects remain in the system but are no longer accessible to the user (per registration-roles spec).
- What happens when a very long project description is submitted? The system should enforce a maximum length (e.g., 5000 characters for description, 200 for title) and display validation errors.
- What happens when two Super Admins try to approve/reject the same project simultaneously? The first decision wins — the second Super Admin sees a message: "This project has already been reviewed."

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide a structured project creation form with predefined fields accessible only to Org Admin users.
- **FR-002**: Mandatory project fields MUST include: title, thematic area, geographic location, lifecycle status, requested funding, implementer type, and description.
- **FR-003**: Taxonomy fields (thematic area, stakeholder sector, implementer type) MUST only allow selection from predefined values. Free-text entry for these fields MUST be rejected.
- **FR-004**: System MUST allow assigning multiple predefined taxonomy values where the field supports it (e.g., multiple thematic areas).
- **FR-005**: System MUST prevent form submission if any required field is missing, with clear validation messages.
- **FR-006**: System MUST save newly created projects with status "Pending Approval."
- **FR-007**: System MUST provide a "My Projects" view for Org Admins listing all their projects with title, thematic area, location, funding requested, and status.
- **FR-008**: Project status indicators MUST include: Draft, Pending Approval, Approved, Rejected, Archived.
- **FR-009**: System MUST provide a publish action with a confirmation modal before changing project status to "Pending Approval."
- **FR-010**: Projects with status "Pending Approval" MUST NOT be visible in public search results or to visitors.
- **FR-011**: System MUST allow Org Admins to edit projects they own, with the same structured form fields as creation.
- **FR-012**: System MUST allow Org Admins to delete projects they own, with a confirmation modal before deletion.
- **FR-013**: Deleted projects MUST be removed from search results, dashboards, and all public views.
- **FR-014**: System MUST provide a Super Admin review queue showing all projects with status "Pending Approval."
- **FR-015**: Super Admins MUST be able to approve a project (changes status to "Approved," becomes publicly visible) or reject it (changes status to "Rejected," with a mandatory reason).
- **FR-016**: When a project is rejected, the system MUST notify the project owner with the rejection reason.
- **FR-017**: Rejected projects MUST be editable and re-publishable by the Org Admin (re-enters "Pending Approval" status).
- **FR-018**: System MUST generate audit log entries for all project lifecycle actions: creation, edit (with field-level diff), deletion, publish, approve, reject.
- **FR-019**: Audit log entries MUST include: who performed the action, what changed, and when.
- **FR-020**: Predefined taxonomy values MUST be seeded from a client-provided list and MUST NOT be editable through the application UI.
- **FR-021**: Assigned taxonomy tags MUST appear on project cards in search results and on project detail pages.
- **FR-022**: Only Org Admin users MUST be able to create, edit, and delete projects. Funders and visitors MUST NOT have access to these actions.
- **FR-023**: System MUST enforce field length limits: title (max 200 characters), description (max 5000 characters).

### Key Entities

- **Project**: A climate finance initiative. Key attributes: id, title, description, thematic area(s), geographic location, lifecycle status, requested funding, implementer type, stakeholder sector, organization name, status (Draft / Pending Approval / Approved / Rejected / Archived), owner (Org Admin user), created at, updated at.
- **Predefined Taxonomy**: Classification values managed outside the application. Key attributes: category (thematic area, stakeholder sector, implementer type, region, lifecycle status), value, display order. Cannot be created or edited via UI.
- **Project Audit Log**: Records project lifecycle events. Key attributes: project id, user id, action type (created, edited, published, approved, rejected, deleted), changed fields (for edits), reason (for rejections), timestamp.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Org Admins can complete the project creation form and submit in under 5 minutes.
- **SC-002**: 100% of submitted projects go through the approval workflow — no project becomes publicly visible without Super Admin approval.
- **SC-003**: Super Admins can review and make a decision (approve/reject) on a project within 2 minutes of opening it.
- **SC-004**: Rejected projects include a reason visible to the project owner in 100% of cases.
- **SC-005**: All project lifecycle events (create, edit, delete, publish, approve, reject) are captured in the audit log with no gaps.
- **SC-006**: Taxonomy tags assigned to projects appear correctly in search filters — visitors can filter by any taxonomy dimension and get accurate results.
- **SC-007**: The "My Projects" page loads within 2 seconds showing all of the Org Admin's projects with correct statuses.
- **SC-008**: The project creation form works correctly on desktop, tablet, and mobile devices.

## Scope

### In Scope
- Project creation form with structured fields and predefined taxonomy selection
- Project publishing with confirmation and "Pending Approval" status
- My Projects list for Org Admins with status indicators
- Project editing with the same structured form
- Project deletion with confirmation
- Super Admin approval/rejection workflow with rejection reasons
- Re-submission of rejected projects
- Predefined taxonomy seeding and enforcement
- Taxonomy tag display on project cards and detail pages
- Audit logging for all project lifecycle actions
- RBAC enforcement (Org Admin only for CRUD, Super Admin for approval)

### Out of Scope
- Project detail page for public visitors (assumed to exist from Phase 1 or as a simple read-only view)
- File/document upload on projects (deferred)
- Project collaboration (multiple Org Admins on one project) — deferred
- Project archiving as a separate action (currently handled by status changes)
- Bulk project operations (approve/reject multiple at once) — deferred
- Project versioning / change history UI (audit log captures data but no UI for viewing history)
- Notification system for approval/rejection (US-16 covers notifications — this spec focuses on the rejection reason being available to the owner, delivery mechanism is out of scope)
- Dashboard metrics based on project data (separate feature)

## Assumptions

- Registration & Roles (002-registration-roles) is implemented — Org Admin and Super Admin roles exist with RBAC enforcement.
- Homepage search (001-homepage) exists — approved projects appear in search results.
- The project data model from Phase 1 includes basic fields (title, description, status). This feature extends it with: structured taxonomy fields, lifecycle status, implementer type, audit trail.
- Predefined taxonomy values will be provided by the client as a seed data file before development begins.
- Email notification for rejection is a simple message — the full notification system (in-app + email) is a separate feature.
- The project detail page for public viewing exists from Phase 1 in basic form.
