# Data Model: Onboarding

**Feature**: 005-onboarding | **Date**: 2026-03-17

## Entities

### UserPreferences (new)

Stores a user's onboarding answers for personalization.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| user_id | String | FK to User (unique — one-to-one) | Required, unique |
| areas_of_interest | Text | Free-text: thematic focus areas | Optional, max 2000 chars |
| geographic_preferences | Text | Free-text: regions/countries of interest | Optional, max 2000 chars |
| funding_focus | Text | Free-text: funding priorities or project types | Optional, max 2000 chars |
| is_onboarding_completed | Boolean | Whether the user submitted onboarding (vs skipped) | Default: false |
| completed_at | DateTime | When onboarding was completed | Nullable |
| created_at | DateTime | When the record was created | Auto-generated |
| updated_at | DateTime | Last update timestamp | Auto-generated |

**Relationships:**
- UserPreferences belongs to User (one-to-one)

**Constraints:**
- `user_id` is unique — one preferences record per user.
- Record is created when user submits onboarding OR when they save preferences from account settings.
- If user skips onboarding, no record is created until they explicitly save preferences later.
- All text fields are optional — a user can answer some questions and leave others blank.

**Indexes:**
- `idx_user_preferences_user_id` on `user_id` (unique)

---

## Derived Data (no new table)

### Recommendation

Recommendations are computed at query time, not stored. They match user preferences against active project data.

**Recommendation shape (response only):**

| Field | Type | Description |
|-------|------|-------------|
| projectId | String | Matched project ID |
| title | String | Project title |
| organizationName | String | Organization |
| thematicAreas | String[] | Project thematic areas |
| location | String | Location |
| shortDescription | String | Brief description |
| matchScore | Number | How well the project matches preferences (0-1) |
| matchReasons | String[] | Why this project was recommended (e.g., "Matches your interest in Renewable Energy") |

---

## No New Enums

This feature uses existing entities and enums from previous features.
