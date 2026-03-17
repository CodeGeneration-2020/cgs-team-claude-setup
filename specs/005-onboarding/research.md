# Research: Onboarding

**Feature**: 005-onboarding | **Date**: 2026-03-17

## R1: Preference Storage Model

**Decision**: Store preferences as structured free-text fields in a `user_preferences` table (one-to-one with User), not as selected taxonomy values.

**Rationale**: The PRD (US-16) specifies "free-text input fields" for onboarding answers. Storing as free text preserves the natural language the user provided and allows future AI-enhanced matching (US-31 Matchmaking). Forcing taxonomy selections at onboarding creates friction and limits expressiveness.

**Alternatives Considered**:
- **Taxonomy multi-select**: More structured, easier to match, but limits user expression and contradicts PRD requirement for free-text. Can be added as an enhancement later.
- **JSON blob**: Flexible but harder to query and validate. Separate text columns are simpler.

---

## R2: Recommendation Algorithm (Initial)

**Decision**: Simple keyword + field matching. Parse user's free-text preferences for keywords that match project taxonomy values (thematic areas, regions) and text similarity against project descriptions. Rank by match count.

**Rationale**: The spec explicitly states "initial implementation uses simple field matching; AI-enhanced matchmaking is separate US-31." A simple approach delivers value without introducing AI dependencies. Keywords are extracted from free text and matched against project taxonomy fields.

**Alternatives Considered**:
- **AI embedding similarity**: More accurate but adds AI service dependency and latency. Deferred to US-31 Matchmaking.
- **No matching (random recommendations)**: Defeats the purpose of onboarding.

**Implementation Notes**:
- Extract keywords from user's areas_of_interest, geographic_preferences, funding_focus.
- Match against: project.thematic_areas, project.location, project.region, project.description.
- Score = count of matching keywords. Return top N projects ordered by score.
- Only include APPROVED/ACTIVE projects.
- Minimum 3 results if possible; fall back to recently added projects if insufficient matches.

---

## R3: Onboarding Flow Integration

**Decision**: Onboarding is a separate page shown after role selection. The registration flow is: Register → Select Role → Onboarding (skippable) → Dashboard.

**Rationale**: Keeping onboarding as a separate step (not embedded in registration) keeps the registration form simple and fast. The onboarding page has its own route so users who skip can return later via account settings.

**Alternatives Considered**:
- **Embed in registration form**: Makes registration longer, increases drop-off.
- **Show on first login instead of after registration**: Users might miss it. After registration is the natural moment of engagement.

---

## R4: Recommendations Placement

**Decision**: Add a "Recommended for you" section on the homepage, above the trending section but below the search bar. For users who haven't completed onboarding, show a prompt card instead.

**Rationale**: The homepage is the most visited page. Placing recommendations there maximizes visibility. The prompt card for non-onboarded users gently encourages completion without being intrusive.

**Alternatives Considered**:
- **Separate recommendations page**: Less visible, users need to navigate to find it.
- **Dashboard-only**: Only visible after login, misses the homepage engagement opportunity.
