# Feature Specification: Project Map

**Feature Branch**: `008-project-map`
**Created**: 2026-03-17
**Status**: Draft
**Input**: User description: "Project Map — small location map on project detail pages showing the geographic area relevant to the project"

## User Scenarios & Testing *(mandatory)*

### US1 — View Project Location on Map (Priority: P1)

As a Visitor, I want to see a small map on a project's detail page so that I can understand the project's geographic location at a glance.

**Why this priority**: This is the entire feature — a single, focused enhancement to the project detail page. It's small, self-contained, and delivers immediate value for understanding project geography.

**Independent Test**: Open a project detail page → verify a map is visible showing the project's location → verify the map does not block other content → verify it works on desktop and mobile.

**Acceptance Scenarios**:

1. **Given** a visitor opens a project detail page, **When** the page loads, **Then** they see a small map showing the project's geographic location.
2. **Given** the map is displayed, **When** the visitor looks at it, **Then** the map clearly indicates the geographic area relevant to the project (e.g., a pin or highlighted region).
3. **Given** the map is displayed, **When** the visitor does not interact with it, **Then** they can still understand the project's location — the map is informational, not requiring interaction.
4. **Given** a project has a location field set, **When** the detail page loads, **Then** the map centers on and highlights the appropriate area.
5. **Given** a project does not have sufficient location data for mapping, **When** the detail page loads, **Then** the map section is hidden rather than showing an empty or generic map.
6. **Given** the map is displayed on desktop, **When** the visitor views the page, **Then** the map is positioned to complement (not dominate or block) the main project information.
7. **Given** the map is displayed on a mobile device, **When** the visitor views the page, **Then** the map is visible and properly sized for the mobile viewport.

---

### Edge Cases

- What happens when a project's location is a broad region (e.g., "East Africa") rather than a specific city? The map shows the broader region with an appropriate zoom level — country/region boundary highlighted rather than a pin.
- What happens when the location text cannot be geocoded (e.g., "Global" or "Multiple regions")? The map section is hidden for that project.
- What happens when the map service is temporarily unavailable? The map section is hidden or shows a placeholder: "Map unavailable." The rest of the project detail page functions normally.
- What happens when a project has both a location text and a region taxonomy value? The map uses the most specific geographic data available (location text preferred, region as fallback).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST display a small map on the project detail page showing the project's geographic location.
- **FR-002**: The map MUST clearly indicate the geographic area relevant to the project (pin for specific locations, region highlight for broad areas).
- **FR-003**: The map MUST be viewable without user interaction — it is informational by default.
- **FR-004**: The map MUST NOT block or dominate the main project information (title, description, funding details).
- **FR-005**: The map MUST be visible on both desktop and mobile views of the project detail page.
- **FR-006**: When a project lacks sufficient location data for mapping, the map section MUST be hidden entirely.
- **FR-007**: When the map service is unavailable, the map section MUST degrade gracefully — hidden or showing a placeholder without breaking the page.
- **FR-008**: The map MUST use the project's location field for geocoding. If location is too broad, the region taxonomy value MAY be used as a fallback.

### Key Entities

No new entities are needed. The map uses existing project data:
- **Project.location**: Free-text location field (e.g., "Kenya, East Africa") — primary source for map positioning.
- **Project.region**: Taxonomy value (e.g., "East Africa") — fallback for broad locations.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of project detail pages with valid location data display a map.
- **SC-002**: The map loads within 2 seconds of the project detail page loading.
- **SC-003**: The map occupies no more than 25% of the visible viewport on desktop and no more than 30% on mobile.
- **SC-004**: 0% of pages break when the map service is unavailable — graceful fallback in all cases.
- **SC-005**: The map correctly positions for 95% of projects with specific location data (city/country level).

## Scope

### In Scope
- Small, non-interactive map on the project detail page
- Map centered on project's location (geocoded from location text or region)
- Pin marker for specific locations, region highlight for broad areas
- Responsive sizing for desktop and mobile
- Graceful fallback when location data is missing or map service is unavailable
- Map positioned to complement, not dominate, project content

### Out of Scope
- Interactive map with zoom/pan controls — this is a static informational map
- Global map showing all projects on a single view (deferred)
- Map on project cards in search results (only on detail pages)
- User-editable map pins during project creation (location is text-based)
- Driving directions or distance calculations
- Map clustering for multiple project locations

## Assumptions

- Project Management (003-project-management) is implemented — projects have location and region fields.
- The project detail page exists (from Phase 1 or 003-project-management).
- A map rendering service will be integrated (e.g., a map tile provider). The specific service is a planning-phase decision.
- Location geocoding (text → coordinates) will be handled either client-side or server-side. The approach is a planning-phase decision.
- The map is a read-only visual element — no interactivity beyond basic display.
