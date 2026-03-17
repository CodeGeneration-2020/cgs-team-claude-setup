# Feature Specification: CFC Homepage

**Feature Branch**: `001-homepage`
**Created**: 2026-03-17
**Status**: Draft
**Input**: User description: "Homepage — search-focused landing page with AI search, trending projects, filtering, sorting, structured results, project tags, and admin-managed trending for the CFC platform"

## User Scenarios & Testing *(mandatory)*

### US1 — Search on Homepage (Priority: P1)

As a Visitor, I want to see a search-focused Homepage so that I can immediately start discovering climate projects without creating an account.

**Why this priority**: Search is the core entry point for all users. Without it, visitors have no way to discover projects — this is the single most critical homepage function.

**Independent Test**: Can be fully tested by navigating to the homepage and verifying the search field is visible above the fold. Submitting a query should navigate to search results. No login required.

**Acceptance Scenarios**:

1. **Given** a visitor opens the homepage, **When** the page loads, **Then** they immediately see a search field without scrolling.
2. **Given** a visitor is on the homepage, **When** they type a free-text query into the search field, **Then** the input is accepted and ready for submission.
3. **Given** a visitor has entered a search query, **When** they submit the search, **Then** they are navigated to a results page showing projects related to the query.
4. **Given** a visitor is not logged in, **When** they use the search field, **Then** the search works without requiring account creation or login.
5. **Given** a visitor submits an empty search query, **When** the form is submitted, **Then** the system either prevents submission or shows all available projects as a default result set.

---

### US2 — AI-Powered Search Matching (Priority: P1)

As a Visitor, I want the system to interpret my search intent so that I receive relevant projects even if I don't use exact keywords.

**Why this priority**: Semantic search is what differentiates the platform from a simple keyword directory. It directly impacts whether visitors find what they need.

**Independent Test**: Can be tested by entering natural-language queries (e.g., "clean water projects in East Africa") and verifying results match the intent, not just keywords.

**Acceptance Scenarios**:

1. **Given** a visitor submits a free-text search request, **When** the system processes it, **Then** results reflect the meaning of the request using semantic search logic, not only exact keyword matches.
2. **Given** a search request contains multiple criteria (e.g., "renewable energy in Southeast Asia"), **When** results are returned, **Then** they reflect both the theme and region criteria when relevant data exists.
3. **Given** a search request is ambiguous or too broad (e.g., "help"), **When** the system processes it, **Then** the user is prompted to refine the search with more specific terms.
4. **Given** two visitors submit similar queries using different wording (e.g., "solar power Africa" vs "photovoltaic energy African continent"), **When** results are returned, **Then** both queries return similar sets of relevant results.
5. **Given** no active projects match the search request, **When** results are displayed, **Then** the system shows a clear message: "No relevant active projects found" with a suggestion to broaden the search or adjust filters.

---

### US3 — Structured Search Results Display (Priority: P1)

As a Visitor, I want to view search results in a structured card layout so that I can easily scan and identify relevant projects.

**Why this priority**: Users need to quickly evaluate results without opening each project individually. This is essential for usability of the search feature.

**Independent Test**: Can be tested by running a search and verifying each result card contains all required information fields and a link to the full project page.

**Acceptance Scenarios**:

1. **Given** a visitor has submitted a search query, **When** results are displayed, **Then** they appear in a card layout with each card showing:
   - Project title
   - Organization name
   - Thematic area
   - Location
   - Project status
   - Short description
2. **Given** search results are displayed, **When** the visitor scans the page, **Then** they can visually evaluate projects without opening each one.
3. **Given** a visitor sees a project card in search results, **When** they click "View project," **Then** they are navigated to the project's full detail page.
4. **Given** search results are displayed, **When** the system filters results, **Then** only projects with status "Active" or "Approved" are shown. Projects with status Archived, Rejected, or Inactive are excluded.
5. **Given** a project becomes inactive after a search was performed, **When** the visitor performs a new search, **Then** the now-inactive project is excluded from results.

---

### US4 — Filter Search Results (Priority: P2)

As a Visitor, I want to filter projects by thematic area, region, lifecycle, organization, funding range, and other criteria so that I can focus on relevant domains.

**Why this priority**: Filters refine the AI search results, which is critical for funders who know their specific focus areas. However, the platform is still usable with search alone.

**Independent Test**: Can be tested by running a search, applying individual and combined filters, and verifying the results update correctly for each filter combination.

**Acceptance Scenarios**:

1. **Given** a visitor sees AI search results, **When** they look at the page, **Then** filter controls are visible and available.
2. **Given** a visitor is on the search results page, **When** they select a filter by thematic area, **Then** the results update to show only projects matching that thematic area.
3. **Given** a visitor applies multiple filters simultaneously (e.g., thematic area + region + funding range), **When** the filters are applied, **Then** the results update to show only projects matching all selected criteria.
4. **Given** a visitor has active filters applied, **When** they remove an individual filter, **Then** the results update to reflect the remaining active filters.
5. **Given** a visitor has multiple filters applied, **When** they click "Clear all filters," **Then** all filters are removed and the original AI search results are restored.
6. **Given** a visitor applies a filter that matches no projects, **When** the results update, **Then** the system displays a message indicating no projects match the current filter combination.

**Available Filter Dimensions**:
- Thematic area
- Region
- Project lifecycle status
- Organization / partner
- Funding range
- Stakeholder sector
- Is "Local NGO" (checkbox)

**Functional Notes**:
- Filters are optional and do not affect the initial search.
- Filter options are dynamically populated based on available project data.

---

### US5 — Sort Search Results (Priority: P2)

As a Visitor, I want to sort results by relevance, date, and name so that I can browse efficiently.

**Why this priority**: Sorting enhances the browsing experience but is not required for basic project discovery. It works in tandem with filters.

**Independent Test**: Can be tested by changing the sort option and verifying the order of results changes accordingly without removing filters or re-running the search.

**Acceptance Scenarios**:

1. **Given** a visitor sees AI search results, **When** they look for a sorting option, **Then** a sort control is available with the following options:
   - Relevance to search (default)
   - Date (newest first)
   - Date (oldest first)
   - Alphabetically by name (A-Z)
2. **Given** a visitor changes the sorting option from "Relevance" to "Newest first," **When** the sort is applied, **Then** the results reorder by date with newest projects first.
3. **Given** a visitor has active filters and changes the sort option, **When** the sort is applied, **Then** the filtered results are reordered without removing or resetting any filters.
4. **Given** a visitor changes sorting, **When** the new order is applied, **Then** the AI search is not re-run — only the display order changes.
5. **Given** only one sorting option can be active at a time, **When** the visitor selects a new sort option, **Then** the previous sort option is deselected.

---

### US6 — Trending Projects Section (Priority: P2)

As a Visitor, I want the Homepage to show featured/trending projects so that I can quickly notice what is active and relevant in the climate finance space.

**Why this priority**: Trending projects provide immediate value for first-time visitors who haven't yet searched. It helps drive engagement before the user takes any action.

**Independent Test**: Can be tested by opening the homepage and verifying the trending section displays project cards with the required information, and that clicking a card navigates to the project detail page.

**Acceptance Scenarios**:

1. **Given** a visitor opens the homepage, **When** the page loads, **Then** a "Trending Projects" section is visible alongside the search field.
2. **Given** the trending section is visible, **When** the visitor looks at featured projects, **Then** each project card shows:
   - Project title
   - Organization name
   - Thematic area
   - Current status
   - Amount of needed funds
   - Location
   - Short summary
3. **Given** featured projects are displayed, **When** the visitor compares them to other page content, **Then** they are visually distinct and clearly identifiable as trending/featured.
4. **Given** a visitor sees a trending project card, **When** they click on it, **Then** they are navigated to that project's detailed page.
5. **Given** no projects are currently marked as trending, **When** the homepage loads, **Then** the trending section is either hidden or displays a placeholder message.

---

### US7 — Visual Tags on Projects (Priority: P2)

As a Visitor, I want to see visual tags on projects so that I can quickly understand their thematic area and status at a glance.

**Why this priority**: Tags improve scannability of both search results and the trending section. They support faster decision-making but are a visual enhancement, not a core function.

**Independent Test**: Can be tested by verifying that project cards in search results and the trending section display visual tags, and that the same tags appear on the project detail page.

**Acceptance Scenarios**:

1. **Given** a visitor sees a project in the search results card layout, **When** they look at the card, **Then** visual tags are displayed indicating the project's thematic area and current status.
2. **Given** visual tags are displayed on a project card, **When** the visitor scans multiple cards, **Then** the tags are visually distinct and easy to recognize at a glance (e.g., color-coded, pill-shaped labels).
3. **Given** a visitor opens a project's detail page, **When** the page loads, **Then** the same visual tags displayed on the card are also shown on the detail page.
4. **Given** a project has multiple thematic areas or tags, **When** the card is displayed, **Then** all relevant tags are shown without breaking the card layout.

---

### US8 — Manage Trending Projects (Priority: P3)

As a Super Admin, I want to manage which projects appear in the Trending section so that funders see the most relevant and active projects on the Homepage.

**Why this priority**: Admin management of trending content is important for platform curation but can initially be managed through direct data changes. A dedicated admin UI is a usability improvement.

**Independent Test**: Can be tested by logging in as Super Admin, adding/removing/reordering trending projects, saving changes, and verifying the homepage updates immediately.

**Acceptance Scenarios**:

1. **Given** a Super Admin opens the admin interface, **When** they navigate to the Trending management section, **Then** they see a list of projects currently marked as Trending.
2. **Given** a Super Admin is on the Trending management page, **When** they search for and select an existing project, **Then** the project is added to the Trending list.
3. **Given** a Super Admin sees the Trending list, **When** they remove a project from the list, **Then** the project is no longer marked as Trending.
4. **Given** multiple projects are in the Trending list, **When** the Super Admin reorders them (e.g., via drag-and-drop or position controls), **Then** the display order is updated.
5. **Given** the Super Admin makes changes to the Trending list, **When** they save the changes, **Then** the updated Trending list is immediately reflected on the Homepage for all visitors.
6. **Given** a Super Admin views a project in the Trending list, **When** they check its details, **Then** they can see: project title, organization name, thematic area, funding need, location, and short summary.

---

### Edge Cases

- What happens when the AI search service is temporarily unavailable? The homepage should display an error message and offer a fallback (e.g., browse by category or see trending projects).
- What happens when a visitor searches with special characters or very long input? The system should sanitize input and limit query length (e.g., 500 characters).
- What happens when the trending projects list is empty? The section should be hidden or show a "No trending projects at this time" message.
- What happens when a trending project is archived or deactivated after being added to the list? The project should automatically be removed from the trending display (but remain in the admin list with a "no longer active" indicator).
- What happens when search results return hundreds of projects? Results should be paginated with a reasonable page size (e.g., 12-20 projects per page).
- What happens when a filter combination returns zero results? A clear "No projects match your filters" message is shown with a suggestion to broaden criteria.
- What happens when a visitor applies filters and then modifies the search query? Filters should be preserved when the query changes, unless the visitor explicitly clears them.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST display a prominent search field on the homepage that is visible without scrolling on standard screen sizes.
- **FR-002**: System MUST allow visitors to use the search, filters, and browse trending projects without authentication.
- **FR-003**: System MUST process free-text search queries using semantic search logic that interprets user intent beyond exact keyword matching.
- **FR-004**: System MUST return search results only for projects with status "Active" or "Approved." Projects with status Archived, Rejected, or Inactive MUST be excluded.
- **FR-005**: System MUST display search results in a card layout showing: project title, organization name, thematic area, location, project status, and short description.
- **FR-006**: System MUST provide filter controls on the search results page for: thematic area, region, project lifecycle status, organization/partner, funding range, stakeholder sector, and "Local NGO" checkbox.
- **FR-007**: System MUST support applying multiple filters simultaneously. Applying or removing a filter MUST update the results without re-running the AI search.
- **FR-008**: System MUST provide sort controls with options: relevance to search, date (newest/oldest), and alphabetically by name. Sorting MUST NOT remove active filters.
- **FR-009**: System MUST display a Trending Projects section on the homepage showing curated projects selected by a Super Admin.
- **FR-010**: System MUST display visual tags on project cards and detail pages indicating thematic area and project status.
- **FR-011**: System MUST provide a Super Admin interface to add, remove, reorder, and save the Trending Projects list.
- **FR-012**: System MUST update the homepage Trending section immediately when a Super Admin saves changes.
- **FR-013**: System MUST paginate search results when the result set exceeds the page size limit.
- **FR-014**: System MUST display a clear message when a search returns no results, with a suggestion to broaden the search or adjust filters.
- **FR-015**: System MUST prompt the user to refine their query when the search request is too broad or ambiguous.
- **FR-016**: System MUST sanitize search input and limit query length to prevent abuse.
- **FR-017**: System MUST automatically exclude deactivated or archived projects from the trending display, even if they remain on the admin's trending list.
- **FR-018**: Filter options MUST be dynamically populated based on available project data values.

### Key Entities

- **Project**: A climate finance initiative submitted by an Organization Admin. Key attributes: title, organization name, thematic area(s), geographic location, lifecycle status, funding requested, short description/summary, visual tags, active/approved status.
- **Trending List**: An ordered collection of projects curated by a Super Admin for homepage display. Key attributes: project reference, display order, date added.
- **Search Query**: A free-text natural-language request submitted by a visitor. Key attributes: query text, timestamp, associated result set.
- **Filter Set**: A combination of filter values applied to narrow search results. Key attributes: thematic area, region, lifecycle status, organization, funding range, stakeholder sector, Local NGO flag.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 90% of visitors can locate a relevant project within 60 seconds of landing on the homepage.
- **SC-002**: Search results appear within 3 seconds of query submission for 95% of requests.
- **SC-003**: Similar queries phrased differently (e.g., "solar energy Africa" vs. "photovoltaic power African continent") return at least 70% overlapping results.
- **SC-004**: Visitors using filters narrow their results to fewer than 20 projects in at least 80% of filter interactions.
- **SC-005**: The trending section is visible without scrolling on desktop and within one scroll on mobile.
- **SC-006**: Super Admin changes to the trending list are reflected on the homepage within 30 seconds of saving.
- **SC-007**: Homepage loads completely (search field interactive, trending visible) within 2 seconds on a standard broadband connection.
- **SC-008**: The homepage is fully functional for unauthenticated visitors — no feature requires login.

## Scope

### In Scope
- Homepage layout with search field and trending projects section
- AI-powered semantic search with natural language processing
- Structured card-based search results display
- Multi-criteria filtering (thematic area, region, lifecycle, organization, funding range, stakeholder sector, Local NGO)
- Sorting by relevance, date, and name
- Visual project tags (thematic area, status)
- Super Admin trending project management interface
- Pagination of search results
- Status-based filtering of results (only Active/Approved)

### Out of Scope
- AI Assistant / guided chatbot (separate feature)
- Project creation, editing, or publishing (separate feature)
- User registration, login, or onboarding (separate feature)
- Bookmarking projects (separate feature)
- Notifications (separate feature)
- Public data dashboard / funding gap charts (separate feature)
- Project map on detail pages (separate feature)
- Matchmaking ranking system (separate feature)
- Dashboard data export (separate feature)

## Assumptions

- The CFC platform has an existing project data model from Phase 1 with fields including title, organization, thematic area, location, status, funding information, and description.
- An AI/semantic search service will be integrated (e.g., vector embeddings or LLM-based search). The specific service is a planning-phase decision.
- Project statuses from Phase 1 include at least: Draft, Pending Approval, Active/Approved, Rejected, Archived, Inactive.
- The Super Admin role exists in the system from Phase 1 with authentication and basic admin access.
- Predefined taxonomy values for thematic areas, regions, and stakeholder sectors will be provided by the client.
- The homepage is a public page accessible without authentication.
- Mobile responsiveness is expected (the homepage must work on desktop, tablet, and mobile).
