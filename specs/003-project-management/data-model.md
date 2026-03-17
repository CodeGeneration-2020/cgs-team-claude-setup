# Data Model: Project Management

**Feature**: 003-project-management | **Date**: 2026-03-17

## Entities

### Project (existing — extended)

The Project entity exists from Phase 1 and was referenced in 001-homepage. This feature extends it with structured form fields, taxonomy references, approval workflow fields, and audit support.

**Existing fields (from Phase 1 + 001-homepage):**

| Field | Type | Description |
|-------|------|-------------|
| id | String (CUID) | Primary key |
| title | String | Project title |
| short_description | String | Brief summary for cards |
| organization_name | String | Organization name |
| location | String | Geographic location |
| status | Enum (ProjectStatus) | Project lifecycle status |
| funding_requested | Decimal | Funding amount needed |
| created_at | DateTime | Creation timestamp |
| updated_at | DateTime | Last update timestamp |

**New/extended fields (Phase 2 — this feature):**

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| description | Text | Full project description | Required, max 5000 chars |
| owner_id | String | FK to User (Org Admin who created it) | Required |
| thematic_areas | String[] | References to taxonomy values (THEMATIC_AREA category) | Required, at least one, must be valid taxonomy values |
| stakeholder_sector | String | Reference to taxonomy value (STAKEHOLDER_SECTOR) | Optional, must be valid taxonomy value |
| implementer_type | String | Reference to taxonomy value (IMPLEMENTER_TYPE) | Required, must be valid taxonomy value |
| lifecycle_status | String | Reference to taxonomy value (LIFECYCLE_STATUS) | Required, must be valid taxonomy value |
| region | String | Reference to taxonomy value (REGION) | Optional, must be valid taxonomy value |
| is_local_ngo | Boolean | Whether implementer is a local NGO | Default: false |
| timeline | String | Project timeline description | Optional, max 1000 chars |
| partners | String | Partner organizations | Optional, max 2000 chars |
| sustainability_info | String | Sustainability plan | Optional, max 2000 chars |
| rejection_reason | String | Reason for rejection (set by Super Admin) | Nullable, set on rejection |
| reviewed_by | String | FK to User (Super Admin who approved/rejected) | Nullable |
| reviewed_at | DateTime | When the approval decision was made | Nullable |
| published_at | DateTime | When the project was first published | Nullable |
| version | Integer | Optimistic lock version for concurrent updates | Default: 1, incremented on update |

**Indexes:**
- `idx_projects_owner_id` on `owner_id` for "My Projects" queries
- `idx_projects_status` on `status` for approval queue and search filtering
- `idx_projects_created_at` on `created_at` for sorting

**Relationships:**
- Project belongs to User (owner, many-to-one)
- Project reviewed by User (reviewer, many-to-one, nullable)

---

### TaxonomyValue (new)

Stores predefined classification values. Seeded from client data. Not editable via UI.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| category | Enum (TaxonomyCategory) | Classification category | Required |
| value | String | The taxonomy value | Required |
| display_order | Integer | Order within the category for UI dropdowns | Default: 0 |
| is_active | Boolean | Whether this value is currently selectable | Default: true |
| created_at | DateTime | When seeded | Auto-generated |

**Constraints:**
- Unique constraint on (category, value) — no duplicate values within a category.
- Values are seeded and managed via database seeds, not the application UI.
- `is_active` allows soft-deprecation of values without breaking existing projects.

**Indexes:**
- `idx_taxonomy_values_category` on `category`
- `idx_taxonomy_values_category_value` on `(category, value)` unique

---

### ProjectAuditLog (new)

Records all project lifecycle actions for accountability and compliance.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| project_id | String | FK to Project (nullable — project may be deleted) | Optional |
| user_id | String | FK to User who performed the action | Required |
| action | Enum (ProjectAction) | Type of action | Required |
| changed_fields | JSON | For edits: diff of old → new values | Nullable |
| metadata | JSON | Additional context (e.g., snapshot on deletion, rejection reason) | Nullable |
| created_at | DateTime | When the action occurred | Auto-generated |

**Constraints:**
- Append-only — entries are never updated or deleted.
- `project_id` is nullable because the project may be deleted, but the audit trail must persist.
- For edits, `changed_fields` stores `{ "field_name": { "old": "value1", "new": "value2" } }`.
- For deletions, `metadata` stores a snapshot of the project at time of deletion.

**Indexes:**
- `idx_project_audit_log_project_id` on `project_id`
- `idx_project_audit_log_user_id` on `user_id`
- `idx_project_audit_log_created_at` on `created_at`

---

## Enum Definitions

### ProjectStatus (extended)

| Value | Description | Visible to Public |
|-------|-------------|-------------------|
| DRAFT | Created but not published | No |
| PENDING_APPROVAL | Published, awaiting Super Admin review | No |
| APPROVED | Approved by Super Admin, publicly visible | Yes |
| REJECTED | Rejected by Super Admin with reason | No |
| ARCHIVED | Removed from active listings by owner | No |

### TaxonomyCategory

| Value | Description |
|-------|-------------|
| THEMATIC_AREA | Climate theme (e.g., Renewable Energy, Clean Water, Biodiversity) |
| STAKEHOLDER_SECTOR | Sector (e.g., Energy, Agriculture, Health) |
| IMPLEMENTER_TYPE | Type of implementing org (e.g., NGO, Government, Private Sector) |
| REGION | Geographic region (e.g., East Africa, Southeast Asia) |
| LIFECYCLE_STATUS | Project stage (e.g., Concept, Planning, Implementation, Scaling) |

### ProjectAction

| Value | Description |
|-------|-------------|
| CREATED | Project created by Org Admin |
| EDITED | Project fields modified |
| PUBLISHED | Project submitted for approval |
| APPROVED | Project approved by Super Admin |
| REJECTED | Project rejected by Super Admin |
| DELETED | Project deleted by Org Admin |
| REPUBLISHED | Rejected project edited and re-submitted |

---

## State Transitions

```
[New Project]
     │
     │ create (save as draft or publish immediately)
     v
  ┌────────┐     publish      ┌──────────────────┐
  │ DRAFT  │ ───────────────> │ PENDING_APPROVAL  │
  └────────┘                  └────────┬──────────┘
                                       │
                              ┌────────┴────────┐
                              │                 │
                          approve           reject
                              │                 │
                              v                 v
                       ┌──────────┐      ┌──────────┐
                       │ APPROVED │      │ REJECTED │
                       └──────┬───┘      └────┬─────┘
                              │               │
                          archive        edit + republish
                              │               │
                              v               v
                       ┌──────────┐   ┌──────────────────┐
                       │ ARCHIVED │   │ PENDING_APPROVAL  │
                       └──────────┘   └──────────────────┘
```

**Transition Rules:**
- DRAFT → PENDING_APPROVAL (publish)
- PENDING_APPROVAL → APPROVED (Super Admin approves)
- PENDING_APPROVAL → REJECTED (Super Admin rejects with reason)
- REJECTED → PENDING_APPROVAL (Org Admin edits + re-publishes)
- APPROVED → ARCHIVED (Org Admin archives)
- Any status → DELETED (Org Admin deletes, hard delete with audit snapshot)
- DRAFT → DELETED (Org Admin deletes draft)
