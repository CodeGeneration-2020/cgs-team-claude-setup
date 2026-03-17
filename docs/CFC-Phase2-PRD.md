# Product Requirements Document: CFC Platform — Phase 2

**Project:** Climate Finance Connector (CFC)
**Phase:** 2 — Additional Scope
**Date:** 2026-03-17
**Source:** CFC Phase 2 User Stories (CSV export)
**Status:** Draft

---

## 1. Executive Summary

Phase 2 of the CFC Platform extends the existing platform with AI-powered search and matchmaking, a public data dashboard, structured project management for organizations, funder engagement features (bookmarks, notifications), role-based access control, onboarding personalization, and platform stabilization measures. The goal is to transform CFC from a static project directory into an intelligent matchmaking platform that connects funders with climate projects efficiently.

---

## 2. Goals & Objectives

1. **AI-Powered Discovery** — Enable funders and visitors to find relevant climate projects through natural language search and guided AI assistant interactions.
2. **Structured Project Management** — Provide Organization Admins with tools to create, edit, publish, and manage project listings with approval workflows.
3. **Funder Engagement** — Give funders bookmark, notification, and matchmaking tools to stay connected with relevant projects.
4. **Data Transparency** — Offer public dashboards showing ecosystem trends, funding gaps, and thematic distributions.
5. **Platform Quality** — Establish role-based access control, structured logging, automated testing, and AI governance guardrails.

---

## 3. User Roles

| Role | Description |
|------|-------------|
| **Visitor** | Unauthenticated user browsing the platform. Can search, view projects, and access public dashboards. |
| **New User** | User in the registration process, choosing their role. |
| **Org Admin** | Registered user who manages an organization's projects. Can create, edit, publish, and delete projects. |
| **Funder** | Registered user looking for projects to fund. Can bookmark projects, receive notifications, and use matchmaking. |
| **Super Admin** | Platform administrator who approves/rejects projects and manages trending content. |
| **Platform Admin** | Technical administrator responsible for system configuration, monitoring, and AI governance. |

---

## 4. Epics & Feature Requirements

### 4.1 Homepage

#### US-1: Search on Homepage
**Role:** Visitor
**Story:** I want to see a search-focused Homepage so that I can immediately start discovering projects.

**Acceptance Criteria:**
1. When user opens the homepage, they immediately see a search field without scrolling.
2. User can type a free-text query into the search field.
3. When user submits their search, they are taken to a page that shows projects related to the query.
4. User can use the search without creating an account or logging in.

**Design Status:** Done

---

#### US-2: Trending Projects
**Role:** Visitor
**Story:** I want the Homepage to show featured/trending projects so that I can quickly notice what is active and relevant.

**Acceptance Criteria:**
1. When user opens the homepage, they can see a section with highlighted projects.
2. User can visually distinguish featured projects from the rest of the page content.
3. For each featured project, user can see basic information:
   - Project title
   - Organization name
   - Thematic area
   - Current status
   - Amount of needed funds
   - Location
   - Short summary
4. When user clicks on a featured project, they are taken to its detailed project page.

**Design Status:** Done

---

#### US-3: Manage Trending Projects
**Role:** Super Admin
**Story:** I want to set Trending projects so that Funders can see them on the Homepage.

**Acceptance Criteria:**
1. When a Super Admin opens the admin interface, they can see a list of projects currently marked as Trending.
2. The Super Admin can add an existing project to the Trending list.
3. The Super Admin can remove a project from the Trending list.
4. The Super Admin can change the display order of Trending projects.
5. For each Trending project, the Super Admin can view: project title, organization name, thematic area, funding need, location, short summary.
6. When the Super Admin saves changes, the updated Trending list is immediately reflected on the Homepage for visitors.

**Design Status:** Not done

---

### 4.2 Search

#### US-4: AI Search
**Role:** Visitor
**Story:** I want to search for projects using free-text so that I can express my needs naturally.

**Acceptance Criteria:**
1. User can see an AI search input on the Homepage.
2. User can type a natural-language request into the search input.
3. User can submit a request to run the AI search.
4. After submitting, user is taken to the Current projects part of the Homepage.
5. User can see a list of project results.
6. User can refine a request using a search field or filters and submit again to update the results.

**Design Status:** Done

---

#### US-5: Search Inquiry Matching
**Role:** Visitor
**Story:** I want the system to interpret my search intent so that I receive relevant projects even if I don't use exact keywords.

**Acceptance Criteria:**
1. When a user submits a free-text search request, the system interprets the request using semantic search logic.
2. The search results reflect the meaning of the request, not only exact keyword matches.
3. If the search request contains multiple criteria (e.g., theme + region), the results reflect those criteria when relevant data exists.
4. If the request is ambiguous or too broad, the system prompts the user to refine the search.
5. Similar search requests written using different wording return similar sets of relevant results.
6. If no active projects match the request, the system displays a message informing the user that no relevant active projects were found.

**Design Status:** Not designed

---

#### US-6: Search Results Display (Status Filtering)
**Role:** Visitor
**Story:** I want to see a list of projects that correspond to my search inquiry so that I can choose the ones I am interested in.

**Acceptance Criteria:**
1. When search results are displayed, the system only includes projects with status "Active" or "Approved."
2. Projects with status Archived, Rejected, or Inactive are excluded from the results.
3. When a user opens a project from the search results, the project page confirms that the project is currently active.
4. If a project becomes inactive after the search is performed, it is not displayed in future search results.

**Design Status:** Not designed

---

#### US-7: Structured Search Results
**Role:** Visitor
**Story:** I want to view AI search results in a structured way so that I can easily identify the relevant information.

**Acceptance Criteria:**
1. User sees AI search results displayed in a card layout.
2. For each project in the results, user can see key information:
   - Project title
   - Organization name
   - Thematic area
   - Location
   - Project status
   - Short description
3. User can visually scan results without opening each project.
4. User can click "View project" to go to the project's full page.

**Design Status:** Not done

---

### 4.3 Filtering, Sorting & Results

#### US-8: Filter Projects
**Role:** Visitor
**Story:** I want to filter projects by thematic area, region, lifecycle, organization, funding range etc, so that I can focus on relevant domains.

**Acceptance Criteria:**
1. User can apply filters to the AI search.
2. User can filter projects by:
   - Thematic area
   - Region
   - Project lifecycle status
   - Organization/partner
   - Funding range
   - Stakeholder sector
   - Is "Local NGO" (checkbox)
3. When user applies one or more filters, the list of results updates accordingly.
4. User can apply multiple filters at the same time.
5. User can remove individual filters.
6. User can clear all filters and return to the original AI search results.

**Functional Requirements:**
- Filters are optional.
- Filters are available on the page with AI search results.
- Filter options are based on available project data.

**Design Status:** Done

---

#### US-9: Sort Results
**Role:** Visitor
**Story:** I want to sort results by relevance, date, name, and match quality so that I can browse efficiently.

**Acceptance Criteria:**
1. After user sees AI search results, user can choose a sorting option.
2. User can sort results by:
   - Relevance to search
   - Date (newest/oldest first)
   - Alphabetically by name
3. When user changes the sorting option, the order of the results updates accordingly.
4. User can change the sorting option without re-running the AI search.
5. Sorting does not remove or reset any filters.

**Functional Requirements:**
- "Relevance to my search" is based on AI relevance scoring.
- Sorting options are mutually exclusive (one active at a time).

**Design Status:** Not done

---

### 4.4 AI Assistant

#### US-10: AI Assistant Guided Questions
**Role:** Visitor
**Story:** I want the assistant to ask me guided questions so that it can understand my interests.

**Acceptance Criteria:**
1. User can open the AI assistant from the widget on the homepage.
2. When user starts interacting with the AI assistant, they are guided through a sequence of predefined questions.
3. The questions help clarify interests, such as focus areas or types of projects.
4. User can answer each question using free text.
5. User can skip a question and continue the onboarding flow.
6. User can stop the onboarding flow at any time.
7. As user progresses through the questions, the assistant responds in a way that reflects their answers.
8. When user finishes or exits the onboarding flow, they can view project search results related to their inputs.
9. Onboarding answers are stored and reused over time during interaction with the platform.

**Design Status:** In progress

---

### 4.5 Project Tags & Indicators

#### US-11: Visual Tags on Projects
**Role:** Visitor
**Story:** I want to see visual tags on projects so that I can quickly understand their type and status.

**Acceptance Criteria:**
1. When user sees a project in a list or card view, they can see visual tags associated with the project.
2. The tags help user quickly understand the project's thematic area and current status.
3. The tags are visually distinct and easy to recognize at a glance.
4. When user opens a project's detail page, they can also see the same visual tags.

**Design Status:** Done

---

### 4.6 Project Map

#### US-12: Project Location Map
**Role:** Visitor
**Story:** I want to see a small map on a project page so that I can understand its location.

**Acceptance Criteria:**
1. When user opens a project detail page, they can see a small map showing the project's location.
2. The map clearly indicates the geographic area relevant to the project.
3. User can view the project location without interacting with the map.

**Functional Requirements:**
- The map does not block or dominate the main project information.
- The map is visible on both desktop and mobile views.

**Design Status:** Done

---

### 4.7 Public Data Dashboard

#### US-13: Dashboard Charts
**Role:** Visitor
**Story:** I want to view ecosystem trends, project distribution by theme, funding trends, and funding gaps.

**Acceptance Criteria:**
- **Global charts:** funding gap, thematic distribution, distribution by stakeholder type, geographic distribution.
- **Per-project charts:** progress bar for funding stage, project lifecycle, radar for project focus.

**Design Status:** Done
**Reference:** [Dashboard specification document](https://docs.google.com/document/d/18tcNVXXxdINNgK54xtYVdP53dliImS--WRG-bpZlP-I/edit?usp=sharing)

---

#### US-29: Funding Gap Calculation Logic
**Role:** Visitor
**Story:** I want funding gaps to be calculated automatically so that I can identify underfunded sectors and regions.

**Acceptance Criteria:**
1. System calculates total requested funding per theme.
2. System calculates total received funding per theme.
3. Funding gap = requested - received.
4. Metrics update when filters are applied.
5. Archived projects are excluded from funding calculations.

**Design Status:** TBD

---

#### US-30: Dashboard Data Export
**Role:** Visitor / Funder
**Story:** I want to export dashboard data so that I can use it in reports.

**Acceptance Criteria:**
1. User can export visible dashboard data as CSV.
2. User can export dashboard charts as PDF.
3. Export respects active filters.
4. Export excludes sensitive/private fields.

**Design Status:** TBD

---

### 4.8 Registration & Onboarding

#### US-14: Registration — Basic Information
**Role:** New User
**Story:** I want to input personal information so that I can create an account.

**Acceptance Criteria:**
1. When a new user opens the registration page, they can enter the required personal information.
2. Required fields include:
   - Name
   - Email address
   - Password
   - Company name
   - Company type
3. The system validates that:
   - The email format is correct
   - The password meets security requirements
4. If validation fails, the user receives a clear error message.
5. When registration is successful, a new user account is created.
6. After successful registration, the user is redirected to the next step of onboarding or role selection.

---

#### US-15: Registration — Select User Type
**Role:** New User
**Story:** I want to choose my role during registration (Organization or Funder) so that I get the corresponding experience for my role.

**Acceptance Criteria:**
1. During registration, the user is prompted to select their account type:
   - Organization Admin
   - Funder
2. The user must select one role before continuing.
3. The selected role determines the features available to the user after registration.
4. After registration is completed, the user is redirected to the appropriate onboarding flow based on their selected role.
5. The selected role is stored in the user profile.

**Design Status:** Not done

---

#### US-16: Onboarding
**Role:** Org Admin / Funder
**Story:** I want to answer the onboarding questions so that the system can personalize my experience.

**Acceptance Criteria:**
1. After registration or account conversion, the user is prompted to complete onboarding questions.
2. The onboarding questions allow the user to describe:
   - Areas of interest
   - Geographic preferences
   - Funding priorities or project focus
3. The user can provide answers using free-text input fields.
4. The user can skip onboarding and proceed directly to the platform.
5. If the user completes onboarding, the system uses the answers to generate personalized project suggestions.
6. When the user returns to the platform later, the system continues to use the saved onboarding data to personalize project recommendations.

**Design Status:** Not done

---

### 4.9 Project Management

#### US-17: Create a Project
**Role:** Org Admin
**Story:** I want to create a project using a structured project form so that I can publish my funding needs on the platform.

**Acceptance Criteria:**
1. When a user logged in as Org Admin opens the project section, they can create a new project.
2. The system displays a structured project creation form with predefined fields.
3. Mandatory fields must include:
   - Thematic area
   - Geographic location
   - Lifecycle status
   - Requested funding
   - Implementer type
4. Thematic areas, sectors, and tags are selected from predefined taxonomy values.
5. The user cannot submit the form if required fields are missing.
6. When the user clicks Save/Publish project, the system validates the form.
7. If validation succeeds, the project is saved with status "Pending Approval."
8. After submission, the project appears in My Projects for the Org Admin.
9. Every project creation action generates an audit log entry (who created and when).

**Design Status:** Not done

---

#### US-18: Publish a Project
**Role:** Org Admin
**Story:** I want to publish a project so that it becomes visible to potential funders.

**Acceptance Criteria:**
1. When the project form is completed, the user can click "Publish Project."
2. A confirmation modal appears: "Publish this project?"
3. The user can choose: Yes, publish / Cancel.
4. When confirmed: the project status becomes "Pending Approval."
5. The project is added to the Admin review queue.
6. The project is not visible publicly until approved by Super Admin.
7. The project owner can see the status "Pending Approval" in their dashboard.

---

#### US-19: Edit a Project
**Role:** Org Admin
**Story:** I want to edit an existing project so that I can update project information if something changes.

**Acceptance Criteria:**
1. When the Org Admin opens My Projects, they see a list of projects they created.
2. The user can select a project and open the Edit Project screen.
3. The system displays the same structured fields used in project creation.
4. The user can modify editable fields including: description, funding information, timeline, tags, partners, sustainability information.
5. Mandatory fields must still be present when saving edits.
6. When the user clicks "Save changes," the system updates the project.
7. If the project is already approved, the system flags the update for review if critical fields changed (optional depending on scope).
8. Each edit action creates an audit log entry capturing: who edited, what fields changed, timestamp.

---

#### US-20: Delete a Project
**Role:** Org Admin
**Story:** I want to delete a project so that I can remove incorrect or duplicate submissions.

**Acceptance Criteria:**
1. When user views a project they own, they see the "Delete project" option.
2. When user selects Delete, a confirmation modal appears: "Are you sure you want to delete this project?"
3. The user can choose: Delete / Cancel.
4. If the user confirms deletion: the project is removed from the system.
5. Deleted projects no longer appear in search results or dashboards.
6. The deletion event is recorded in audit logs.

---

#### US-21: View My Projects
**Role:** Org Admin
**Story:** I want to see all projects I created so that I can manage them easily.

**Acceptance Criteria:**
1. Org Admin can access a "My Projects" section.
2. The page shows a list of projects created by the user.
3. For each project the user can see: project title, thematic area, location, funding requested, project status.
4. Project status indicators include: Draft, Pending Approval, Approved, Rejected, Archived.
5. The user can open each project to: edit, delete, view details.

---

#### US-22: Approve/Reject a Project
**Role:** Super Admin
**Story:** I want to review and approve or reject submitted projects so that only verified projects are published.

**Acceptance Criteria:**
1. User can see a list of projects pending review.
2. User can open a submitted project and review its details.
3. User can approve a project.
4. User can reject a project with a reason.
5. When user approves or rejects a project, the project status changes accordingly.
6. When a project is approved, visitors can view it on the platform.
7. When a project is rejected, the project owner is notified with the reason.

**Design Status:** TBD
**Open Question:** Do we have this functionality in the current system?

---

### 4.10 Bookmarks

#### US-23: Add Bookmark
**Role:** Funder
**Story:** I want to bookmark projects so that I can revisit them later.

**Acceptance Criteria:**
1. When user views a project, they can choose to bookmark it.
2. User can see that a project has been bookmarked.
3. User can remove a bookmark from a project.
4. Bookmarking or removing a bookmark does not interrupt browsing.
5. Any bookmarks remain saved when user leaves and returns to the platform.

**Design Status:** Done

---

#### US-24: View My Bookmarks
**Role:** Funder
**Story:** I want to view my bookmarked projects so that I can easily review projects I am interested in.

**Acceptance Criteria:**
1. When user bookmarks a project, they can be informed about related updates.
2. User can see updates related to their bookmarked projects in "My Account" area.
3. Each update clearly indicates which project it relates to.
4. User is notified about all significant changes on project description, status, stakeholders, funding stage.
5. User can open a bookmarked project directly from an update.
6. If user removes a bookmark from a project, they no longer receive updates about it.

**Design Status:** In progress

---

### 4.11 Notifications

#### US-25: Email Notifications
**Role:** Funder
**Story:** I want to receive email notifications about changes in my Bookmarked projects so that I stay informed without constantly checking the platform.

**Acceptance Criteria:**
1. Users receive notifications when bookmarked projects change.
2. Notifications are stored in a notification center.
3. Notifications can be marked as read.
4. Notification events are logged.
5. Email notification is supported (if in scope).

**Design Status:** Not designed

---

#### US-26: In-App Notifications
**Role:** Funder
**Story:** I want to have a dedicated Notifications page on the platform so that I can see changes in my Bookmarked projects.

**Acceptance Criteria:**
1. Users receive in-app notifications for:
   - Bookmarked project updates
   - Project status changes
   - Major description updates
   - New projects strongly matching their onboarding profile
2. Notifications appear in a Notification Center.
3. Notifications can be marked as read.
4. Users may receive email notifications for the same events.
5. Users can opt out of email notifications while still receiving in-app notifications.
6. Notification events are logged for system auditing.

---

### 4.12 Account Management

#### US-27: Convert Account Type
**Role:** Org Admin
**Story:** I want to convert my account into a funder account so that I can use funder-specific features on the platform.

**Acceptance Criteria:**
1. As an existing user without active projects, user can see an option to convert their account to a funder account.
2. User can choose to start the account conversion.
3. After conversion, user can access funder-specific features.
4. After conversion, user no longer sees options related to managing projects.
5. Any existing account credentials remain valid.

**Design Status:** Not done

---

#### US-28: Opt Out of Email Notifications
**Role:** Funder
**Story:** I want to opt out of email notifications so that I no longer receive them.

**Acceptance Criteria:**
1. Users can access Notification Settings from the user account menu.
2. The settings page contains a control for Email Notifications.
3. Users can disable email notifications using a toggle or checkbox.
4. When email notifications are disabled: the user stops receiving email notifications.
5. Disabling email notifications does not affect in-platform notifications.
6. The system saves the user's notification preference in their profile.
7. Users can re-enable email notifications at any time.

---

### 4.13 Matchmaking

#### US-31: Matchmaking Ranking Logic
**Role:** Funder
**Story:** I want projects to be ranked based on how well they match my interests so that I can focus on the most relevant opportunities.

**Acceptance Criteria:**
1. Projects are ranked using structured field matching (theme, region, funding gap, etc.).
2. Ranking integrates AI relevance score (if applicable).
3. Ranking logic is consistent and reproducible.
4. Archived projects are excluded.

**Design Status:** Not designed

---

### 4.14 Roles & Access Control

#### US-32: Role-Based Access Control
**Role:** Platform Admin
**Story:** I want features to be restricted by user role so that each user sees only relevant functionality.

**Acceptance Criteria:**
1. Organizational Admin can manage projects.
2. Funder cannot edit projects.
3. Visitors cannot access restricted features.
4. Permissions are enforced on backend level.

**Design Status:** Not designed

---

### 4.15 Predefined Taxonomy & Classification

#### US-33: Predefined Classification Model
**Role:** System
**Story:** The system must use predefined classification fields so that filtering and dashboard calculations remain consistent.

**Acceptance Criteria:**
1. Classification fields are seeded from client-provided list.
2. Tags cannot be created or edited via admin UI.
3. Only predefined values can be assigned to projects.
4. Structured fields are usable in filtering and dashboard aggregation.

**Design Status:** Not designed

---

#### US-34: Assign Predefined Tags to Project
**Role:** Org Admin
**Story:** I want to assign predefined classification values to a project so that it appears correctly in search and dashboard.

**Acceptance Criteria:**
1. User can select classification values during project creation/editing.
2. User can assign multiple predefined values where allowed.
3. Invalid values are rejected.
4. Assigned values appear in project cards and detail pages.

**Design Status:** Not designed

---

### 4.16 AI Governance

#### US-35: AI Data Protection
**Role:** Platform Admin
**Story:** I want AI processing to exclude sensitive data so that user privacy is protected.

**Acceptance Criteria:**
1. AI requests include only public project fields.
2. Personal contact data is never sent to AI service.
3. Financial private fields are excluded.
4. API keys are stored securely on backend.

**Design Status:** Not designed

---

#### US-36: AI Usage Logging
**Role:** Platform Admin
**Story:** I want AI usage to be logged so that platform costs can be monitored.

**Acceptance Criteria:**
1. AI requests are logged at backend level as part of structured logging.

**Design Status:** Not designed

---

### 4.17 Platform Stabilization

#### US-37: Structured Logging & Monitoring
**Role:** Platform Admin
**Story:** I want system errors and performance issues to be logged and monitored so that technical problems can be detected and resolved quickly.

**Acceptance Criteria:**
1. Backend errors are logged in a structured format.
2. Logs include timestamp, request ID, and context.
3. Critical errors are captured by monitoring service.
4. The system exposes a health-check endpoint.
5. Sensitive user data is not logged.

**Design Status:** Not designed

---

#### US-38: Automated Regression Testing
**Role:** Platform Admin
**Story:** I want automated regression and smoke tests so that new releases do not break existing functionality.

**Acceptance Criteria:**
1. Core user flows are covered by automated tests.
2. Tests run automatically during deployment.
3. Deployment fails if critical tests fail.
4. Test coverage meets agreed minimum threshold.

**Design Status:** Not designed

---

## 5. Design Status Summary

| Status | Count | User Stories |
|--------|-------|-------------|
| **Done** | 8 | US-1, US-2, US-4, US-8, US-11, US-12, US-13, US-23 |
| **In progress** | 2 | US-10, US-24 |
| **Not done** | 7 | US-3, US-7, US-9, US-15, US-16, US-17, US-27 |
| **Not designed** | 12 | US-5, US-6, US-25, US-31, US-32, US-33, US-34, US-35, US-36, US-37, US-38, US-34 |
| **TBD** | 3 | US-22, US-29, US-30 |
| **No status** | 6 | US-14, US-18, US-19, US-20, US-21, US-26, US-28 |

---

## 6. Open Questions

1. **US-22 (Project Approval):** Does the current system already have project approval/rejection functionality? (Flagged by BA)
2. **US-29 (Funding Gap Calculation):** Does this functionality exist in the current system? (Flagged by BA)
3. **US-30 (Dashboard Export):** Does this functionality exist in the current system? (Flagged by BA)
4. **US-7 vs US-6:** Search result display (US-7) and status filtering (US-6) may need to be combined or clarified for implementation — they describe overlapping aspects of the same search results page.
5. **US-10 (AI Assistant):** How does the AI assistant onboarding relate to US-16 (Onboarding)? Are they separate flows or should they converge?
6. **US-27 (Account Conversion):** What happens to existing project data when an Org Admin converts to Funder? The AC says "without active projects" — what about archived/rejected projects?

---

## 7. Dependencies & Assumptions

### Dependencies
- AI/semantic search service (OpenAI, Anthropic, or similar) for US-4, US-5, US-10, US-31
- Map service (Mapbox, Google Maps, or similar) for US-12
- Email delivery service (SendGrid, AWS SES, or similar) for US-25
- Client-provided taxonomy/classification list for US-33, US-34
- Monitoring service (Sentry, Datadog, or similar) for US-37

### Assumptions
- Phase 1 of the platform is live with basic project listing and viewing.
- User authentication exists from Phase 1.
- Project data model exists from Phase 1 and will be extended.
- The predefined taxonomy list will be provided by the client before US-33/US-34 development.

---

## 8. Priority Grouping

### P1 — Core (must-have for Phase 2 launch)
- **Search:** US-1, US-2, US-4, US-5, US-6, US-7, US-8, US-9
- **Project Management:** US-17, US-18, US-19, US-20, US-21, US-22
- **Registration & Roles:** US-14, US-15, US-32
- **Taxonomy:** US-33, US-34

### P2 — Engagement (high value, can follow shortly after)
- **AI Assistant:** US-10
- **Bookmarks & Notifications:** US-23, US-24, US-25, US-26, US-28
- **Onboarding:** US-16
- **Trending:** US-3
- **Tags & Map:** US-11, US-12

### P3 — Analytics & Advanced
- **Dashboard:** US-13, US-29, US-30
- **Matchmaking:** US-31
- **Account Conversion:** US-27

### P4 — Platform Quality
- **AI Governance:** US-35, US-36
- **Stabilization:** US-37, US-38
