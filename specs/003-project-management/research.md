# Research: Project Management

**Feature**: 003-project-management | **Date**: 2026-03-17

## R1: Project Form Structure

**Decision**: Use a single, unified project form component for both creation and editing. The form uses controlled components with client-side validation matching server-side DTO validation rules.

**Rationale**: Create and edit use the same fields (FR-011). A single form component reduces duplication and ensures consistency. Pre-filling values for edit mode is a prop-level concern.

**Alternatives Considered**:
- **Separate create/edit forms**: Duplicate code, higher maintenance. Rejected.
- **Multi-step wizard form**: Over-engineered — the form has ~10 fields, not dozens. A single-page form with sections is sufficient.

---

## R2: Taxonomy Storage & Enforcement

**Decision**: Store predefined taxonomy values in a `taxonomy_values` table with `category` and `value` columns. Seed from a JSON/CSV file. Validate project taxonomy fields against the table at the service layer.

**Rationale**: FR-003 and FR-020 require taxonomy values to be predefined, not user-editable, and enforced on all project forms. A database table enables: validation at the service layer, populating dropdown options via API, and future extensibility (adding categories or values via seeds without code changes).

**Alternatives Considered**:
- **Hardcoded enums**: Simpler but requires code deployment to add values. Rejected per FR-020 (seeded from client list).
- **JSON config file**: No database query needed, but harder to validate and can't be queried for filter options. Rejected.

**Implementation Notes**:
- Categories: THEMATIC_AREA, STAKEHOLDER_SECTOR, IMPLEMENTER_TYPE, REGION, LIFECYCLE_STATUS.
- The taxonomy API endpoint (GET /v1/taxonomy) returns values grouped by category for frontend dropdowns.
- Project creation/edit DTOs validate that provided values exist in the taxonomy table.

---

## R3: Audit Log Design

**Decision**: Use a dedicated `project_audit_log` table (separate from the auth audit log in 002-registration-roles). Each entry records: project id, user id, action type, changed fields (JSON diff for edits), reason (for rejections), and timestamp.

**Rationale**: FR-018/FR-019 require comprehensive audit logging with field-level change tracking. A separate table keeps project auditing independent from auth auditing and allows project-specific queries.

**Alternatives Considered**:
- **Single unified audit log**: Simpler schema but mixes concerns, harder to query project history specifically.
- **Event sourcing**: Full history of every change, but over-engineered for the current requirement (we need logging, not replay).

**Implementation Notes**:
- For edits: compute diff between old and new values, store as JSON in `changed_fields` column.
- For deletions: capture a snapshot of the project data at deletion time in `metadata`.
- Audit log writes are fire-and-forget (non-blocking) to avoid impacting user-facing response times.

---

## R4: Approval Workflow

**Decision**: Simple two-state approval: Pending Approval → Approved or Rejected. No multi-step approval chain. Rejected projects can be edited and re-published.

**Rationale**: The PRD describes a straightforward approve/reject flow (US-22). Multi-step approval (e.g., reviewer → approver) is not mentioned and would add complexity without clear value.

**Alternatives Considered**:
- **Multi-step approval**: Multiple reviewers required. Not in PRD, deferred.
- **Auto-approval for trusted orgs**: Useful for established organizations. Not in scope, can be added later.

**Implementation Notes**:
- Re-publishing a rejected project: Org Admin edits the project, clicks "Publish," status returns to "Pending Approval."
- Concurrent approval handling: Use optimistic locking — if two Super Admins try to decide on the same project, the first decision wins and the second sees "already reviewed."

---

## R5: Project Deletion Strategy

**Decision**: Hard delete with audit trail. The project record is removed from the database, but a deletion audit log entry captures a snapshot of the project data.

**Rationale**: FR-013 states "Deleted projects MUST be removed from search results, dashboards, and all public views." Hard delete is the simplest way to guarantee this. The audit log preserves the historical record for accountability.

**Alternatives Considered**:
- **Soft delete (is_deleted flag)**: Keeps data but requires filtering on every query. Adds complexity to search, dashboard, and list queries. Could leak into results if a filter is missed.
- **Archive status**: Similar to soft delete. The spec treats Archive as a separate status from deletion.

---

## R6: Rejection Notification

**Decision**: For this feature, store the rejection reason on the project record and make it visible when the Org Admin views the project. Full email/in-app notification is out of scope (deferred to the Notifications feature).

**Rationale**: FR-016 says "the system MUST notify the project owner with the rejection reason." The spec's Out of Scope section says "Notification system for approval/rejection (US-16 covers notifications)." So we make the reason visible in-app on the project detail / My Projects list, and the notification delivery mechanism is handled by the separate Notifications feature.

**Implementation Notes**:
- Store `rejection_reason` (nullable string) and `reviewed_by` (FK to user) on the project entity.
- Org Admin sees a banner/message on rejected projects in "My Projects" with the reason and option to edit + re-publish.
