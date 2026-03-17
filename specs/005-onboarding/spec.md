# Feature Specification: Onboarding

**Feature Branch**: `005-onboarding`
**Created**: 2026-03-17
**Status**: Draft
**Input**: User description: "Onboarding — personalized post-registration questionnaire for Org Admins and Funders to describe interests, geographic preferences, and funding priorities for tailored project recommendations"

## User Scenarios & Testing *(mandatory)*

### US1 — Complete Onboarding Questionnaire (Priority: P1)

As a newly registered Org Admin or Funder, I want to answer onboarding questions so that the system can personalize my experience with relevant project recommendations.

**Why this priority**: Onboarding data drives personalization — without it, all users see the same generic experience. Collecting preferences is the foundation for matchmaking, tailored search, and future notifications about relevant new projects.

**Independent Test**: Register a new account → select role → presented with onboarding questions → answer them → verify answers are saved → verify personalized project suggestions appear.

**Acceptance Scenarios**:

1. **Given** a user has just completed registration and role selection, **When** they proceed to the next step, **Then** they are prompted to complete onboarding questions.
2. **Given** the onboarding questionnaire is displayed, **When** the user reads the questions, **Then** they can describe:
   - Areas of interest (thematic focus)
   - Geographic preferences (regions they care about)
   - Funding priorities or project focus (what kind of projects they want to find or fund)
3. **Given** the user is answering onboarding questions, **When** they provide answers, **Then** they can use free-text input fields to express their preferences naturally.
4. **Given** the user has answered all questions, **When** they submit the onboarding form, **Then** their answers are saved to their profile.
5. **Given** the user completes onboarding, **When** they are redirected to the platform, **Then** the system uses their answers to show personalized project suggestions (e.g., on a dashboard or as a "Recommended for you" section).
6. **Given** the user returns to the platform later, **When** they browse or search, **Then** the system continues to use their saved onboarding data to personalize recommendations.

---

### US2 — Skip Onboarding (Priority: P1)

As a newly registered user, I want to skip onboarding and go straight to the platform so that I can start using it immediately without answering questions.

**Why this priority**: Mandatory onboarding creates friction and increases drop-off. Users must be able to skip it and still use the platform fully. This is essential for a good registration flow.

**Independent Test**: Register → presented with onboarding → click "Skip" → verify redirect to platform → verify platform works normally without onboarding data.

**Acceptance Scenarios**:

1. **Given** a user is presented with the onboarding questionnaire, **When** they look at the screen, **Then** they see a clearly visible "Skip" option (e.g., "Skip for now" link or button).
2. **Given** a user clicks "Skip," **When** the action is processed, **Then** they are immediately redirected to the platform (dashboard or homepage) without saving any onboarding data.
3. **Given** a user skipped onboarding, **When** they use the platform, **Then** all features work normally — search, browse, bookmarks (for Funders), project creation (for Org Admins). No features are blocked.
4. **Given** a user skipped onboarding, **When** they view search results, **Then** results are not personalized (default relevance ranking, no "Recommended for you" section).

---

### US3 — Update Onboarding Answers (Priority: P2)

As a registered user, I want to update my onboarding answers later so that the system adjusts recommendations as my interests change.

**Why this priority**: Interests evolve over time. Users who skipped onboarding may want to complete it later, and users who completed it may want to refine their answers. However, this can be delivered after the initial onboarding flow works.

**Independent Test**: Log in → navigate to account settings → find "My Preferences" section → edit onboarding answers → save → verify updated recommendations.

**Acceptance Scenarios**:

1. **Given** a user is logged in (with or without previous onboarding), **When** they navigate to their account settings, **Then** they see a "My Preferences" or "Personalization" section.
2. **Given** a user opens the preferences section, **When** the page loads, **Then** they see their existing onboarding answers pre-filled (or empty fields if they skipped).
3. **Given** a user modifies their answers, **When** they save the changes, **Then** the updated answers are persisted to their profile.
4. **Given** a user has updated their preferences, **When** they browse the platform next, **Then** recommendations reflect the updated preferences.
5. **Given** a user who skipped onboarding opens preferences, **When** they fill in the fields and save, **Then** the system treats this as completing onboarding — personalization is now active.

---

### US4 — Personalized Project Suggestions (Priority: P2)

As a user who completed onboarding, I want to see project suggestions tailored to my interests so that I can find relevant projects faster.

**Why this priority**: This is the payoff of onboarding — without personalized suggestions, there's no visible reason to complete the questionnaire. However, the onboarding flow itself must work first.

**Independent Test**: Complete onboarding with specific interests → navigate to dashboard/homepage → verify "Recommended for you" section shows projects matching stated interests.

**Acceptance Scenarios**:

1. **Given** a user has completed onboarding with specific interests (e.g., "Renewable Energy" + "East Africa"), **When** they visit the homepage or dashboard, **Then** they see a "Recommended for you" section with projects that match their stated interests.
2. **Given** the recommendations section is displayed, **When** the user reviews the suggested projects, **Then** each project's relevance to their interests is apparent (matching thematic area, region, or focus).
3. **Given** a user has not completed onboarding (skipped), **When** they visit the homepage, **Then** the "Recommended for you" section is either hidden or shows a prompt: "Complete your preferences to get personalized recommendations."
4. **Given** a user's onboarding includes geographic preferences, **When** recommendations are generated, **Then** projects in matching regions are prioritized.
5. **Given** recommendations are shown, **When** the user clicks on a recommended project, **Then** they are navigated to the project's detail page.

---

### Edge Cases

- What happens when no projects match the user's onboarding preferences? Show a message: "No projects currently match your preferences. Try broadening your interests or check back later."
- What happens when a user completes onboarding with very broad preferences (e.g., all regions, all themes)? The system treats this similar to having no preferences — recommendations are less targeted but still weighted.
- What happens when onboarding data becomes stale (user set preferences a year ago)? The system uses whatever data exists. The "Update preferences" option in account settings is available for users to refresh.
- What happens when the user closes the browser during onboarding? Partially filled answers are lost. The user can complete onboarding later via account settings (US3).
- What happens during account conversion (Org Admin → Funder)? Onboarding data is preserved — preferences are role-agnostic (interests, regions, focus areas apply to both roles).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST present an onboarding questionnaire after registration and role selection.
- **FR-002**: The onboarding questionnaire MUST include questions about: areas of interest, geographic preferences, and funding priorities or project focus.
- **FR-003**: Users MUST be able to answer questions using free-text input fields.
- **FR-004**: Users MUST be able to skip onboarding entirely and proceed directly to the platform. No feature MUST be blocked by incomplete onboarding.
- **FR-005**: System MUST save onboarding answers to the user's profile when submitted.
- **FR-006**: System MUST use saved onboarding data to generate personalized project suggestions.
- **FR-007**: Users MUST be able to view and update their onboarding answers from account settings at any time.
- **FR-008**: When a user updates their preferences, the system MUST use the updated data for future recommendations.
- **FR-009**: System MUST display a "Recommended for you" section on the homepage or dashboard for users who have completed onboarding.
- **FR-010**: The recommendations section MUST show projects matching the user's stated interests (thematic areas, regions, focus).
- **FR-011**: For users who skipped onboarding, the recommendations section MUST either be hidden or show a prompt to complete preferences.
- **FR-012**: System MUST preserve onboarding data during account conversion (Org Admin → Funder).
- **FR-013**: System MUST display a "No matching projects" message when no projects match the user's preferences.
- **FR-014**: Onboarding questions MUST be the same for both Org Admin and Funder roles (preferences are role-agnostic).

### Key Entities

- **UserPreferences**: Stores a user's onboarding answers. Key attributes: user id, areas of interest (text), geographic preferences (text), funding/project focus (text), onboarding completed flag, completed at timestamp, updated at timestamp. One-to-one relationship with User.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 60% of new users complete the onboarding questionnaire (not forced — measured as voluntary completion rate).
- **SC-002**: Users can complete the onboarding questionnaire in under 3 minutes.
- **SC-003**: Users who completed onboarding find a relevant project 40% faster than users who skipped (measured by time from login to first project view or bookmark).
- **SC-004**: Personalized recommendations show at least 3 relevant projects for 80% of users who completed onboarding.
- **SC-005**: 100% of users who skip onboarding can access all platform features without restriction.
- **SC-006**: Onboarding preferences can be updated from account settings within 30 seconds.

## Scope

### In Scope
- Post-registration onboarding questionnaire (3 questions: interests, geography, focus)
- Free-text input for all questions
- Skip onboarding option
- Save onboarding answers to user profile
- Update preferences from account settings
- "Recommended for you" section based on onboarding data
- Empty state for no-match recommendations
- Prompt for users who haven't completed onboarding
- Preservation of preferences during account conversion

### Out of Scope
- AI Assistant guided questionnaire (separate feature — US-10, different interaction model)
- Onboarding for Visitors (unauthenticated users — cannot save preferences)
- Complex preference matching algorithms (initial implementation uses simple field matching; AI-enhanced matchmaking is separate US-31)
- Mandatory onboarding — always skippable
- Progressive profiling (learning from user behavior over time) — deferred
- Multi-language onboarding questions — deferred

## Assumptions

- Registration & Roles (002-registration-roles) is implemented — users can register and select roles.
- Homepage search (001-homepage) exists — recommendations can integrate alongside search results.
- Projects have thematic areas, regions, and other taxonomy fields (from 003-project-management) that can be matched against preferences.
- The initial recommendation algorithm is simple: match user's free-text preferences against project taxonomy fields and descriptions. AI-enhanced matching is a future feature (US-31 Matchmaking).
- The same onboarding questions apply to both Org Admins and Funders — preferences are about interests, not role-specific workflows.
