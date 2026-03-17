# Feature Specification: AI Assistant

**Feature Branch**: `007-ai-assistant`
**Created**: 2026-03-17
**Status**: Draft
**Input**: User description: "AI Assistant — guided chatbot widget on homepage that asks predefined questions to understand user interests and shows matching project results"

## User Scenarios & Testing *(mandatory)*

### US1 — Guided Question Flow (Priority: P1)

As a Visitor, I want the AI assistant to ask me guided questions so that it can understand my interests and help me find relevant projects.

**Why this priority**: The guided flow is the core value proposition — it turns a passive search experience into an interactive discovery tool. Without it, the assistant is just another search box.

**Independent Test**: Open homepage → click AI assistant widget → go through question sequence → answer each question → verify the assistant responds contextually → at the end, see project results matching answers.

**Acceptance Scenarios**:

1. **Given** a visitor is on the homepage, **When** they click the AI assistant widget/icon, **Then** the assistant opens (as a chat panel or overlay) and begins the guided question flow.
2. **Given** the assistant has started, **When** the visitor interacts with it, **Then** they are guided through a sequence of predefined questions designed to clarify their interests (e.g., focus areas, project types, geographic regions).
3. **Given** a question is presented, **When** the visitor answers using free text, **Then** the assistant accepts the input and proceeds to the next question.
4. **Given** the visitor is on a question, **When** they choose to skip it, **Then** the assistant moves to the next question without requiring an answer.
5. **Given** the visitor is mid-flow, **When** they choose to stop/close the assistant, **Then** the flow ends immediately and the assistant closes.
6. **Given** the visitor progresses through questions, **When** the assistant presents the next question, **Then** it responds in a way that reflects or acknowledges the visitor's previous answers (contextual responses).
7. **Given** the visitor finishes or exits the question flow, **When** the flow ends, **Then** they can view project search results related to their inputs.

---

### US2 — Store and Reuse Answers (Priority: P2)

As a registered user, I want my AI assistant answers to be stored so that the platform can use them to personalize my experience over time.

**Why this priority**: Persistence adds long-term value but the assistant works for single sessions without it. Visitors (unauthenticated) get results without storage. Registered users get persistence.

**Independent Test**: Log in → complete assistant flow → log out → log back in → verify the platform remembers answers and uses them for recommendations.

**Acceptance Scenarios**:

1. **Given** an authenticated user completes the assistant question flow, **When** the flow ends, **Then** their answers are saved to their profile.
2. **Given** a user's assistant answers are saved, **When** they return to the platform later, **Then** the system uses their saved answers to personalize project recommendations (integrated with onboarding preferences from 005-onboarding).
3. **Given** a visitor (not logged in) completes the assistant flow, **When** the flow ends, **Then** they see project results for the current session only — answers are not persisted.
4. **Given** a visitor completes the flow and then registers/logs in, **When** they authenticate, **Then** their session answers are transferred to their profile.

---

### US3 — View Results from Assistant (Priority: P1)

As a Visitor, I want to view project results based on my assistant answers so that I can find relevant projects without using the search box directly.

**Why this priority**: Results are the payoff of the guided flow. Without showing results, the questions are pointless. This must ship alongside US1.

**Independent Test**: Complete assistant flow → verify results page shows projects matching stated interests → verify can refine with search/filters.

**Acceptance Scenarios**:

1. **Given** the visitor finishes the assistant question flow, **When** results are displayed, **Then** they see a list of projects related to their answers.
2. **Given** results are displayed from the assistant, **When** the visitor looks at the results, **Then** they are shown in the same card layout as the standard search results (using the existing ProjectCard component).
3. **Given** results are shown, **When** the visitor wants to refine, **Then** they can use the search field or filters to further narrow the results.
4. **Given** the assistant produces no matching projects, **When** results are displayed, **Then** the system shows: "No projects match your interests. Try broadening your answers or search directly."

---

### Edge Cases

- What happens when a visitor opens the assistant and immediately closes it? No data is captured, no results shown. The widget returns to its initial state.
- What happens when the AI service is unavailable? The assistant shows a message: "The assistant is temporarily unavailable. Please use the search bar to find projects." The rest of the platform continues to work.
- What happens when a user provides very short or nonsensical answers? The assistant accepts any free-text input and passes it to the matching logic. Poor-quality answers may yield poor results, but the system doesn't block the user.
- What happens when the assistant has already been completed by a logged-in user? They can restart the flow from the widget, and new answers overwrite previous ones.
- What happens on mobile? The assistant opens as a full-screen overlay rather than a side panel to accommodate smaller screens.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide an AI assistant widget accessible from the homepage.
- **FR-002**: The AI assistant MUST guide the user through a sequence of predefined questions when opened.
- **FR-003**: Questions MUST help clarify user interests such as: focus areas, types of projects, geographic regions, and funding priorities.
- **FR-004**: Users MUST be able to answer each question using free-text input.
- **FR-005**: Users MUST be able to skip any question and continue the flow.
- **FR-006**: Users MUST be able to stop/close the assistant at any time.
- **FR-007**: The assistant MUST respond contextually — each follow-up message should acknowledge or reflect the user's previous answers.
- **FR-008**: When the user finishes or exits the flow, the system MUST display project search results related to their inputs.
- **FR-009**: Results from the assistant MUST use the same card layout and project data as the standard search results.
- **FR-010**: Users MUST be able to refine assistant results using the search field or filters.
- **FR-011**: For authenticated users, assistant answers MUST be saved to their profile and reused for personalization (integrated with onboarding preferences).
- **FR-012**: For unauthenticated visitors, answers MUST NOT be persisted beyond the current session.
- **FR-013**: If a visitor completes the flow and then registers/logs in during the same session, their answers MUST be transferred to the new account.
- **FR-014**: The assistant MUST display a fallback message when the AI service is unavailable.
- **FR-015**: The assistant widget MUST work on desktop (side panel) and mobile (full-screen overlay).
- **FR-016**: The predefined question sequence MUST be configurable without requiring code changes (e.g., stored as data, not hardcoded in UI components).

### Key Entities

- **AssistantSession**: A single interaction session with the AI assistant. Key attributes: session id, user id (nullable for visitors), question-answer pairs, resulting project IDs, started at, completed at. Sessions are transient for visitors, persisted for authenticated users.
- **AssistantQuestion**: A predefined question in the guided flow. Key attributes: question id, question text, display order, is required (or skippable). Configurable without code changes.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 30% of homepage visitors who see the assistant widget engage with it (open and answer at least one question).
- **SC-002**: 70% of users who start the assistant flow complete at least 3 out of the predefined questions.
- **SC-003**: Users who complete the assistant flow find a relevant project 50% faster than those who use the search bar alone (measured by time to first project click or bookmark).
- **SC-004**: Assistant responses appear within 3 seconds of the user submitting an answer.
- **SC-005**: The assistant works without errors for 99% of interactions (measured by completion rate vs error/timeout rate).
- **SC-006**: 100% of visitors can use the assistant without creating an account.

## Scope

### In Scope
- AI assistant widget on the homepage (chat panel / overlay)
- Predefined guided question sequence (configurable)
- Free-text answers to each question
- Skip question capability
- Stop/close assistant at any time
- Contextual responses reflecting previous answers
- Project results display after flow completion
- Integration with existing search results and filters
- Answer persistence for authenticated users (merged with onboarding preferences)
- Session-only results for unauthenticated visitors
- Answer transfer on registration/login
- Mobile-responsive assistant (full-screen overlay)
- Fallback when AI service is unavailable

### Out of Scope
- Open-ended conversational AI (the assistant follows a predefined question flow, not free-form chat)
- Voice interaction
- Multi-language support for assistant questions — deferred
- Assistant appearing on pages other than the homepage — deferred
- Learning from conversation history to improve future responses (deferred to Matchmaking US-31)
- Admin UI for editing questions (configurable via data, but no dedicated admin page)

## Assumptions

- Homepage search (001-homepage) is implemented — assistant results feed into the existing search results display.
- Onboarding (005-onboarding) is implemented — assistant answers for authenticated users are merged into onboarding preferences (UserPreferences entity).
- An AI service is available for generating contextual responses and matching answers to projects. The specific service is a planning-phase decision.
- The predefined questions are similar in nature to the onboarding questions (interests, geography, focus) but presented conversationally rather than as a form.
- The question sequence is short (3-5 questions) to maintain engagement.
- The assistant is a homepage enhancement, not a replacement for the search bar.
