# Research: AI Assistant

**Feature**: 007-ai-assistant | **Date**: 2026-03-17

## R1: LLM Integration for Contextual Responses

**Decision**: Use an LLM API (e.g., Claude or GPT) to generate contextual follow-up messages and to convert the user's free-text answers into a structured search query for project matching.

**Rationale**: FR-007 requires the assistant to respond contextually, reflecting previous answers. Simple template strings can't achieve natural conversation. An LLM generates natural follow-ups and can also extract structured intent (themes, regions, focus areas) from free-text answers to build a search query.

**Alternatives Considered**:
- **Template-based responses**: Predictable but rigid. "Thank you for your answer. Next question:" feels robotic. Rejected.
- **Rule-based NLU**: Complex to build, limited to anticipated patterns. LLM is more flexible.
- **No contextual responses (just next question)**: Simpler but fails FR-007 and feels like a form, not a conversation.

**Implementation Notes**:
- System prompt includes: the current question, previous Q&A pairs, and instructions to be conversational and concise.
- After the final question, the LLM extracts keywords/themes from all answers → used as the search query for project matching.
- LLM calls include only question text and user answers — no PII, no project data sent to LLM (AI Governance US-35).
- Each LLM request is logged with: session id, token count, latency (AI Governance US-36).

---

## R2: Question Storage and Configuration

**Decision**: Store predefined questions in an `assistant_questions` database table with display order, question text, and a "is skippable" flag. Seed via a migration/seed script.

**Rationale**: FR-016 requires questions to be configurable without code changes. Database storage allows updating questions via seed scripts without redeployment.

**Alternatives Considered**:
- **Hardcoded in frontend**: Violates FR-016 (configurable without code changes).
- **JSON config file**: Works but requires deployment to update. Database is more flexible.
- **Admin UI for editing**: Over-engineered — questions change rarely. Seed scripts are sufficient.

---

## R3: Session Management

**Decision**: Create a session record when the assistant starts. For authenticated users, persist the session with answers in the database. For visitors, use a temporary session ID (cookie or in-memory) that expires when the browser closes.

**Rationale**: US2 requires answer persistence for authenticated users but session-only for visitors. A dual approach handles both: database for auth users, ephemeral for visitors.

**Implementation Notes**:
- Start session endpoint returns a session ID.
- Each answer submission includes the session ID.
- On login/register: if a visitor session exists, transfer it to the new user account (FR-013).
- Session stores: question-answer pairs as JSON, resulting search query, completion status.

---

## R4: Answer-to-Search Integration

**Decision**: After the assistant flow completes, redirect the user to the existing search results page (from 001-homepage) with the LLM-extracted query as the search parameter. Reuse the existing search endpoint and results display.

**Rationale**: FR-009 and FR-010 require results to use the same card layout and support filters/search refinement. Rather than building a separate results view, feed the assistant's output into the existing search infrastructure.

**Alternatives Considered**:
- **Separate results display within the chat**: Duplicates the search results UI. More work, less consistent.
- **Custom matching endpoint**: Unnecessary — the existing semantic search (001-homepage) already handles natural-language queries.

---

## R5: Widget UX Pattern

**Decision**: Floating action button (FAB) in the bottom-right corner of the homepage. Opens a chat panel (side drawer on desktop, full-screen overlay on mobile). Panel contains the chat flow with messages, input, and skip/close controls.

**Rationale**: FAB + chat panel is the industry-standard pattern for assistant/chatbot widgets (Intercom, Drift, etc.). Users recognize it immediately. FR-015 requires desktop (side panel) and mobile (full-screen).

**Implementation Notes**:
- Widget shows a subtle animation or label to attract attention without being intrusive.
- Chat panel persists during the session — user can minimize and re-open without losing progress.
- On completion, the panel shows a "View Results" button that navigates to search results.
