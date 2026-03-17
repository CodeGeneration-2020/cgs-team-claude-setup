# Data Model: AI Assistant

**Feature**: 007-ai-assistant | **Date**: 2026-03-17

## Entities

### AssistantQuestion (new)

Predefined questions for the guided flow. Seeded via database, not editable via UI.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| question_text | String | The question displayed to the user | Required, max 500 chars |
| display_order | Integer | Order in the question sequence | Required, unique |
| is_skippable | Boolean | Whether the user can skip this question | Default: true |
| placeholder_text | String | Hint text for the input field | Optional, max 200 chars |
| is_active | Boolean | Whether this question is currently in the flow | Default: true |
| created_at | DateTime | When seeded | Auto-generated |

**Constraints:**
- Unique on `display_order` (no two questions share the same position).
- Only `is_active=true` questions are shown in the flow.
- Managed via seed scripts, not application UI.

**Indexes:**
- `idx_assistant_questions_display_order` on `display_order`

---

### AssistantSession (new)

Records a user's interaction with the assistant. Persisted for authenticated users, ephemeral for visitors.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key / session ID | Auto-generated |
| user_id | String | FK to User (nullable for visitors) | SET NULL, nullable |
| answers | JSON | Array of { questionId, questionText, answerText } | Required after first answer |
| extracted_query | String | Search query extracted from answers by LLM | Nullable, set on completion |
| is_completed | Boolean | Whether the user finished the flow | Default: false |
| started_at | DateTime | When the session started | Auto-generated |
| completed_at | DateTime | When the flow was completed/exited | Nullable |

**Relationships:**
- AssistantSession belongs to User (many-to-one, nullable)

**Constraints:**
- For visitors: session record may be created in-memory only and not persisted to DB. Or persisted with null user_id and cleaned up periodically.
- For authenticated users: persisted and linked to user_id.
- On login/register: if a session with null user_id matches the current browser session, update user_id to the new account.

**Indexes:**
- `idx_assistant_sessions_user_id` on `user_id`

---

## No New Enums

This feature uses existing enums from previous features. The answers JSON structure is schemaless (array of question-answer objects).
