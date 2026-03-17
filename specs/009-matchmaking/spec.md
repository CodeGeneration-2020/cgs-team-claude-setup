# Feature Specification: Matchmaking

**Feature Branch**: `009-matchmaking`
**Created**: 2026-03-17
**Status**: Draft
**Input**: User description: "Matchmaking — AI-powered project ranking for Funders based on their profile, structured field matching (theme, region, funding gap), and AI relevance scoring"

## User Scenarios & Testing *(mandatory)*

### US1 — Ranked Project Matches for Funders (Priority: P1)

As a Funder, I want projects to be ranked based on how well they match my interests so that I can focus on the most relevant funding opportunities.

**Why this priority**: This is the core matchmaking value — transforming a generic project list into a personalized ranking. Without ranking, the Funder must manually evaluate every project.

**Independent Test**: Complete onboarding with specific interests → navigate to a "Matches" section → verify projects are ranked with the most relevant first → verify match quality score is visible.

**Acceptance Scenarios**:

1. **Given** a Funder has completed onboarding preferences (areas of interest, geographic preferences, funding focus), **When** they access their matched projects, **Then** projects are ranked by relevance to their profile — most relevant first.
2. **Given** the ranking is displayed, **When** the Funder views a project in the list, **Then** each project shows a match quality indicator (e.g., percentage or score) explaining how well it matches their profile.
3. **Given** ranked results are displayed, **When** the Funder looks at the match details, **Then** they can see why a project was matched (e.g., "Matches your interest in Renewable Energy," "Located in your preferred region: East Africa").
4. **Given** the ranking logic, **When** projects are scored, **Then** the ranking uses structured field matching (theme, region, funding gap alignment) combined with AI relevance scoring for a composite match score.
5. **Given** a project is archived or inactive, **When** the ranking is generated, **Then** archived/inactive projects are excluded from the matches.
6. **Given** a Funder has not completed onboarding, **When** they access the matches section, **Then** they see a prompt: "Complete your preferences to see personalized project matches."

---

### US2 — Match Consistency and Reproducibility (Priority: P1)

As a Funder, I want the ranking to be consistent so that the same projects appear in the same order when I revisit, unless my preferences or the projects change.

**Why this priority**: Inconsistent rankings erode trust. If projects reshuffle randomly on each visit, users can't rely on the system. This is a quality requirement for US1.

**Independent Test**: View matches → note the order → close and reopen → verify same order. Update preferences → verify ranking changes to reflect new preferences.

**Acceptance Scenarios**:

1. **Given** a Funder views their matched projects, **When** they revisit the same page later (without changing preferences), **Then** the ranking order is the same (deterministic, reproducible).
2. **Given** a Funder updates their onboarding preferences, **When** they view matches after the update, **Then** the ranking reflects the new preferences.
3. **Given** a new project is added that matches a Funder's profile, **When** the Funder views their matches, **Then** the new project appears in the ranking at the appropriate position.
4. **Given** a previously matched project is archived, **When** the Funder views matches, **Then** the archived project is no longer in the list.

---

### US3 — Enhanced Recommendations (Priority: P2)

As a Funder, I want the matchmaking system to improve the existing "Recommended for you" section with AI-powered scoring so that recommendations become more accurate over time.

**Why this priority**: This upgrades the basic keyword matching from onboarding (005-onboarding) to a proper AI-powered system. It's an enhancement, not a new feature surface — it replaces the simple recommendation logic.

**Independent Test**: Compare recommendation quality before and after matchmaking: same user preferences should return more relevant projects with the AI-enhanced scoring.

**Acceptance Scenarios**:

1. **Given** the matchmaking system is active, **When** the homepage "Recommended for you" section loads, **Then** it uses the AI-enhanced matchmaking scoring instead of the basic keyword matching from onboarding.
2. **Given** a Funder has rich preferences (multiple interests, specific regions), **When** recommendations are generated, **Then** the AI scoring produces more relevant results than simple keyword overlap.
3. **Given** the matchmaking system uses both structured fields and AI scoring, **When** results are generated, **Then** the composite score weights structured matches (exact theme/region) higher than fuzzy AI matches.

---

### Edge Cases

- What happens when a Funder's preferences are very narrow and few projects match? Show available matches (even if few) with a message: "Only X projects match your current preferences. Consider broadening your interests."
- What happens when two projects have identical match scores? Use creation date as a tiebreaker (newest first).
- What happens when the AI scoring service is unavailable? Fall back to structured field matching only. The ranking still works, just without the AI relevance boost.
- What happens when a Funder has no preferences at all? Show the prompt to complete onboarding. Do not show random projects as "matches."

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST rank projects for each Funder based on how well they match the Funder's onboarding preferences.
- **FR-002**: The ranking MUST use structured field matching: thematic area overlap, geographic region match, funding gap alignment, stakeholder sector match.
- **FR-003**: The ranking MUST integrate AI relevance scoring that compares Funder's free-text preferences against project descriptions and metadata.
- **FR-004**: The composite match score MUST weight structured matches higher than fuzzy AI matches.
- **FR-005**: Each ranked project MUST display a match quality indicator (score or percentage).
- **FR-006**: Each ranked project MUST display match reasons explaining why it was recommended (e.g., "Matches: Renewable Energy, East Africa").
- **FR-007**: The ranking MUST be consistent and reproducible — same inputs produce same ordering.
- **FR-008**: Archived and inactive projects MUST be excluded from match results.
- **FR-009**: When a Funder has no preferences, the system MUST NOT show ranked matches — display a prompt to complete onboarding instead.
- **FR-010**: The matchmaking ranking MUST replace the basic keyword matching in the existing "Recommended for you" section (from 005-onboarding).
- **FR-011**: When the AI scoring service is unavailable, the system MUST fall back to structured field matching only.
- **FR-012**: Match results MUST be paginated (20 per page).

### Key Entities

- **MatchScore**: A computed score for a Funder-Project pair. Key attributes: funder user id, project id, structured score (0-1), ai score (0-1), composite score (0-1), match reasons (list of strings), computed at. This may be computed on-the-fly or cached.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 80% of Funders who view their top-5 matches find at least 2 relevant projects (measured by click-through or bookmark rate).
- **SC-002**: Match results load within 3 seconds for 95% of requests.
- **SC-003**: Rankings are deterministic — same preferences + same project set = identical ordering 100% of the time.
- **SC-004**: The AI-enhanced matchmaking produces 30% higher click-through rates on recommendations compared to the basic keyword matching it replaces.
- **SC-005**: Fallback to structured-only matching works within 1 second when AI service is unavailable.

## Scope

### In Scope
- Composite ranking: structured field matching + AI relevance scoring
- Match quality score per project (visible to Funder)
- Match reasons per project (why this project was recommended)
- Deterministic, reproducible rankings
- Pagination of match results
- Replacement of basic keyword recommendations (005-onboarding) with AI-enhanced scoring
- Fallback to structured-only when AI is unavailable
- Exclusion of archived/inactive projects
- Prompt for users without preferences

### Out of Scope
- Learning from Funder behavior (clicks, bookmarks) to improve rankings — deferred (requires behavioral data collection)
- Funder-to-Funder matching (connecting Funders with similar interests) — deferred
- Project-to-Funder outreach suggestions (telling Org Admins which Funders might be interested) — deferred
- Real-time score updates (scores are computed on request or cached, not streamed)
- Match score explanation beyond simple reasons (e.g., no detailed breakdown of sub-scores)

## Assumptions

- Onboarding (005-onboarding) is implemented — Funder preferences exist in UserPreferences entity.
- Homepage search (001-homepage) with semantic search embeddings is implemented — AI scoring can leverage the same embedding infrastructure.
- Project taxonomy fields (003-project-management) are populated — structured matching uses thematic areas, regions, sectors.
- The existing "Recommended for you" section (005-onboarding) will be upgraded to use matchmaking scoring. The API endpoint may change or the existing GET /v1/recommendations endpoint is enhanced.
- AI scoring uses the same embedding/LLM infrastructure as homepage semantic search (001-homepage). No new AI service is introduced.
