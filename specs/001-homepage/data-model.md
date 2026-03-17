# Data Model: CFC Homepage

**Feature**: 001-homepage | **Date**: 2026-03-17

## Entities

### Project (existing — extended)

The Project entity is assumed to exist from Phase 1. This feature reads project data for search, display, and filtering. No schema changes to the core project table are required, but the following fields must be present and populated.

**Required fields for Homepage feature:**

| Field | Type | Description | Used By |
|-------|------|-------------|---------|
| id | String (CUID) | Primary key | All |
| title | String | Project title | Card display, search |
| organization_name | String | Name of the submitting organization | Card display, filter |
| thematic_area | String[] | Predefined taxonomy values | Card display, filter, tags |
| location | String | Geographic location/region | Card display, filter |
| status | Enum | Draft, Pending Approval, Active, Approved, Rejected, Archived, Inactive | Status filter, display filter |
| funding_requested | Decimal | Amount of funding needed | Card display, filter (range) |
| short_description | String | Brief summary (max ~300 chars) | Card display |
| stakeholder_sector | String | Predefined taxonomy value | Filter |
| is_local_ngo | Boolean | Whether the implementer is a local NGO | Filter (checkbox) |
| lifecycle_status | String | Project lifecycle stage | Filter |
| created_at | DateTime | Creation timestamp | Sort by date |
| updated_at | DateTime | Last update timestamp | Sort by date |

**Search embedding (new):**

| Field | Type | Description |
|-------|------|-------------|
| embedding | Vector | Semantic embedding of title + description + thematic area + location |

> Note: The embedding may be stored in a separate table or vector index depending on the storage approach chosen during implementation. This is an implementation detail.

---

### TrendingProject (new)

Represents a project curated by a Super Admin for display in the Homepage trending section.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| project_id | String | Foreign key to Project | Required, unique (a project can only be trending once) |
| display_order | Integer | Position in the trending list | Required, >= 0 |
| added_by | String | Foreign key to User (Super Admin who added it) | Required |
| created_at | DateTime | When the project was added to trending | Auto-generated |
| updated_at | DateTime | Last update timestamp | Auto-generated |

**Relationships:**
- TrendingProject belongs to Project (many-to-one)
- TrendingProject belongs to User/Admin (many-to-one, the admin who added it)

**Constraints:**
- project_id is unique — a project can appear in the trending list at most once.
- Only projects with status Active or Approved can be added to trending.
- If a project's status changes to Archived, Rejected, or Inactive, it should be automatically excluded from the public trending display (but the TrendingProject record may remain for admin visibility).

**Indexes:**
- `idx_trending_projects_display_order` on `display_order` for ordered retrieval.
- `idx_trending_projects_project_id` on `project_id` (unique) for lookup.

---

### SearchLog (new, optional)

Captures search queries for analytics and AI usage monitoring.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| query_text | String | The user's search query | Required, max 500 chars |
| result_count | Integer | Number of results returned | Required |
| filters_applied | JSON | Snapshot of filter values applied | Optional |
| sort_option | String | Sort option used | Optional |
| session_id | String | Anonymous session identifier | Optional (no PII) |
| duration_ms | Integer | Time taken to return results | Required |
| created_at | DateTime | Timestamp of the search | Auto-generated |

**Constraints:**
- No personally identifiable information (PII) is stored.
- Query text is stored for analytics only — never includes user identity for anonymous searches.

---

## State Transitions

### Project Status (relevant to Homepage display)

```
                  ┌─────────────┐
                  │    Draft    │
                  └──────┬──────┘
                         │ publish
                         v
              ┌──────────────────────┐
              │   Pending Approval   │
              └──────┬───────┬───────┘
                     │       │
              approve│       │ reject
                     v       v
              ┌──────────┐  ┌──────────┐
              │  Active  │  │ Rejected │
              │ Approved │  └──────────┘
              └──────┬───┘
                     │ archive / deactivate
                     v
              ┌──────────────┐
              │  Archived /  │
              │  Inactive    │
              └──────────────┘
```

**Homepage Display Rules:**
- Only projects with status **Active** or **Approved** appear in search results.
- Only projects with status **Active** or **Approved** appear in the trending section.
- Projects in **Draft**, **Pending Approval**, **Rejected**, **Archived**, or **Inactive** status are excluded from all public homepage displays.

---

## Enum Definitions

### ProjectStatus

| Value | Description |
|-------|-------------|
| DRAFT | Created but not submitted |
| PENDING_APPROVAL | Submitted, awaiting Super Admin review |
| ACTIVE | Approved and publicly visible |
| APPROVED | Approved and publicly visible (alias for Active) |
| REJECTED | Rejected by Super Admin |
| ARCHIVED | Removed from active listings by owner |
| INACTIVE | Deactivated |

### SearchSortOption

| Value | Description |
|-------|-------------|
| RELEVANCE | AI similarity score (default) |
| DATE_NEWEST | Most recently created first |
| DATE_OLDEST | Oldest first |
| NAME_ASC | Alphabetical A-Z by title |
