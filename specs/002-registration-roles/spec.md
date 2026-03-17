# Feature Specification: Registration & Role-Based Access Control

**Feature Branch**: `002-registration-roles`
**Created**: 2026-03-17
**Status**: Draft
**Input**: User description: "Registration & Roles — user signup with role selection (Org Admin / Funder), role-based access control enforcement. Note: some parts of registration already exist from Phase 1."

## User Scenarios & Testing *(mandatory)*

### US1 — Complete Registration with Basic Information (Priority: P1)

As a New User, I want to input my personal and company information so that I can create an account on the platform.

**Why this priority**: Registration is the gateway to all authenticated features. Without it, no user can access project management, bookmarks, or admin functions. Basic registration fields may partially exist from Phase 1 — this story ensures all required fields are present and validated.

**Independent Test**: Navigate to registration page → fill in all required fields → submit → verify account is created and user is redirected to role selection.

**Acceptance Scenarios**:

1. **Given** a new user opens the registration page, **When** the page loads, **Then** they see a form with fields for: name, email address, password, company name, and company type.
2. **Given** a new user fills in all required fields with valid data, **When** they submit the form, **Then** a new account is created and the user is redirected to the role selection step.
3. **Given** a new user enters an email that is already registered, **When** they submit the form, **Then** the system displays an error: "An account with this email already exists."
4. **Given** a new user enters an invalid email format, **When** they submit the form, **Then** the system displays a validation error on the email field.
5. **Given** a new user enters a password that does not meet security requirements, **When** they submit the form, **Then** the system displays a clear message explaining the password requirements (minimum length, complexity).
6. **Given** a new user leaves a required field empty, **When** they submit the form, **Then** the system highlights the missing field(s) with a validation message.
7. **Given** a new user completes registration successfully, **When** the account is created, **Then** the user receives a confirmation email (if email verification is enabled).

---

### US2 — Select User Role During Registration (Priority: P1)

As a New User, I want to choose my role during registration (Organization Admin or Funder) so that I get the corresponding experience for my role.

**Why this priority**: Role selection determines the entire user experience — Org Admins manage projects, Funders browse and bookmark. Without role selection, the platform cannot differentiate user capabilities.

**Independent Test**: After registration → presented with role selection → choose a role → verify the selected role is stored and the user is redirected to the appropriate post-registration flow.

**Acceptance Scenarios**:

1. **Given** a user has just completed the registration form, **When** the account is created, **Then** they are presented with a role selection screen with two options: "Organization Admin" and "Funder."
2. **Given** a user is on the role selection screen, **When** they have not selected a role, **Then** they cannot proceed to the next step (the continue/submit button is disabled or shows a prompt).
3. **Given** a user selects "Organization Admin," **When** they confirm the selection, **Then** the role is stored in their profile and they are redirected to the Org Admin onboarding flow (or dashboard).
4. **Given** a user selects "Funder," **When** they confirm the selection, **Then** the role is stored in their profile and they are redirected to the Funder onboarding flow (or dashboard).
5. **Given** a user has selected a role and completed registration, **When** they log in subsequently, **Then** the platform shows features appropriate for their selected role.

---

### US3 — Role-Based Feature Access (Priority: P1)

As a Platform Admin, I want features to be restricted by user role so that each user sees only the functionality relevant to them and cannot perform unauthorized actions.

**Why this priority**: RBAC is a security requirement that underpins every authenticated feature. Without it, any user could access any endpoint, creating data integrity and security risks.

**Independent Test**: Log in as each role → verify accessible features match the role → attempt to access a restricted feature → verify access is denied with appropriate error.

**Acceptance Scenarios**:

1. **Given** a user is logged in as an Organization Admin, **When** they navigate the platform, **Then** they can access: project creation, project editing, project deletion, "My Projects" list, and their account settings.
2. **Given** a user is logged in as a Funder, **When** they navigate the platform, **Then** they can access: project search, project bookmarks, notifications, and their account settings. They cannot see project creation or editing features.
3. **Given** a user is logged in as a Funder, **When** they attempt to access a project management endpoint directly (e.g., by URL or API call), **Then** the system denies access and returns a "Forbidden" response.
4. **Given** a user is not logged in (Visitor), **When** they attempt to access any authenticated feature (bookmarks, project creation, admin panel), **Then** the system redirects them to the login page or returns an "Unauthorized" response.
5. **Given** a user is logged in as a Super Admin, **When** they navigate the platform, **Then** they can access all features available to Org Admin and Funder, plus admin-specific features (project approval, trending management).
6. **Given** a user's role is stored in their profile, **When** they make any request to a protected endpoint, **Then** the system checks their role against the endpoint's required permission before processing the request.
7. **Given** a user attempts to escalate their role (e.g., modify their role to Super Admin), **When** the request is processed, **Then** the system rejects the modification and logs the attempt.

---

### US4 — User Login (Priority: P1)

As a Registered User, I want to log in with my email and password so that I can access my account and role-specific features.

**Why this priority**: Login is required for any authenticated interaction. It is tightly coupled with registration and RBAC — all three must work together for the platform to function.

**Independent Test**: Navigate to login page → enter valid credentials → verify redirect to appropriate dashboard based on role. Enter invalid credentials → verify error message.

**Acceptance Scenarios**:

1. **Given** a registered user opens the login page, **When** the page loads, **Then** they see fields for email and password, and a submit button.
2. **Given** a registered user enters correct email and password, **When** they submit the form, **Then** they are authenticated and redirected to their role-appropriate dashboard (Org Admin dashboard or Funder dashboard).
3. **Given** a user enters an incorrect password, **When** they submit the form, **Then** the system displays a generic error: "Invalid email or password" (without revealing which field is wrong).
4. **Given** a user enters an email that is not registered, **When** they submit the form, **Then** the system displays the same generic error: "Invalid email or password."
5. **Given** a user has failed to log in multiple times in succession, **When** they exceed the rate limit threshold, **Then** the system temporarily blocks further login attempts and displays a message: "Too many login attempts. Please try again later."
6. **Given** a logged-in user, **When** they click "Log out," **Then** their session is terminated and they are redirected to the homepage.
7. **Given** a user on the login page, **When** they click "Forgot password," **Then** they are directed to the password recovery flow.

---

### US5 — Password Recovery (Priority: P2)

As a Registered User, I want to reset my password if I forget it so that I can regain access to my account.

**Why this priority**: Important for user retention but not blocking for the core registration/login/RBAC flow. Can be delivered after the P1 stories.

**Independent Test**: Click "Forgot password" → enter email → receive reset link → set new password → log in with new password.

**Acceptance Scenarios**:

1. **Given** a user is on the password recovery page, **When** they enter their registered email and submit, **Then** the system sends a password reset link to that email.
2. **Given** a user enters an email that is not registered, **When** they submit the recovery form, **Then** the system displays the same success message (to prevent email enumeration): "If an account exists with this email, a reset link has been sent."
3. **Given** a user clicks the reset link in their email, **When** the link is valid and not expired, **Then** they are taken to a "Set new password" page.
4. **Given** a user clicks an expired or already-used reset link, **When** the page loads, **Then** the system displays: "This reset link has expired. Please request a new one."
5. **Given** a user sets a new password that meets security requirements, **When** they submit the new password, **Then** the password is updated and the user is redirected to the login page with a success message.
6. **Given** a user sets a new password, **When** they log in with the new password, **Then** authentication succeeds.

---

### US6 — Account Conversion (Priority: P3)

As an Organization Admin without active projects, I want to convert my account to a Funder account so that I can use funder-specific features.

**Why this priority**: Edge case for users who change their role on the platform. Not blocking for launch — can be delivered later.

**Independent Test**: Log in as Org Admin with no active projects → navigate to account settings → convert to Funder → verify funder features are now accessible and project management features are hidden.

**Acceptance Scenarios**:

1. **Given** a user is logged in as Org Admin and has no active projects (no projects with status Active, Approved, or Pending Approval), **When** they open account settings, **Then** they see an option to "Convert to Funder account."
2. **Given** a user is logged in as Org Admin and has active projects, **When** they open account settings, **Then** the "Convert to Funder account" option is not visible or is disabled with a message explaining why.
3. **Given** an Org Admin clicks "Convert to Funder account," **When** a confirmation dialog appears and they confirm, **Then** their role is changed to Funder.
4. **Given** a user has converted from Org Admin to Funder, **When** they navigate the platform, **Then** they can access funder features (bookmarks, notifications) and can no longer see project management features.
5. **Given** a user has converted from Org Admin to Funder, **When** they log in again, **Then** their existing credentials still work and their new role is reflected.

---

### Edge Cases

- What happens when a user tries to register with a disposable/temporary email? The system should accept any valid email format — blocking disposable emails is deferred unless explicitly required.
- What happens when the email confirmation link expires? The system should allow the user to request a new confirmation email from the login page.
- What happens when a user's session expires while they are filling out the registration form? The form data should be preserved (client-side) so the user doesn't lose their input.
- What happens when two users try to register with the same email simultaneously? The system should handle the race condition gracefully — the first to complete gets the account, the second sees "email already exists."
- What happens when a Super Admin role needs to be assigned? Super Admin is not self-selectable during registration — it is assigned through a separate internal process (database seed or admin panel).
- What happens when a user tries to access a page they don't have permission for via direct URL? They receive a 403 Forbidden response and are shown a "You don't have permission to access this page" message with a link to their dashboard.
- What happens if a user converts from Org Admin to Funder but still has Draft or Rejected projects? Draft and Rejected projects are retained in the system but become inaccessible to the user. They can be cleaned up by a Super Admin if needed.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide a registration form with required fields: name, email, password, company name, and company type.
- **FR-002**: System MUST validate email format and password strength during registration. Password MUST require minimum 8 characters with at least one uppercase letter, one lowercase letter, one number, and one special character.
- **FR-003**: System MUST prevent duplicate accounts — registration with an already-registered email MUST fail with a clear error.
- **FR-004**: System MUST present a role selection step after registration with two options: Organization Admin and Funder. The user MUST select one before proceeding.
- **FR-005**: System MUST store the selected role in the user's profile and use it to determine feature access throughout the platform.
- **FR-006**: System MUST enforce role-based access control on every protected endpoint:
  - Visitor (unauthenticated): public pages only (homepage, search, trending, project detail)
  - Funder: bookmarks, notifications, account settings, search
  - Organization Admin: project CRUD, "My Projects," account settings, search
  - Super Admin: all of the above + project approval, trending management, admin panel
- **FR-007**: System MUST deny access with a "Forbidden" response when a user attempts to access a feature outside their role's permissions. The denial MUST be enforced at the backend level, not just the UI.
- **FR-008**: System MUST provide a login form with email and password fields.
- **FR-009**: System MUST authenticate users and return a session/token upon successful login. The session MUST include the user's role.
- **FR-010**: System MUST display a generic error message ("Invalid email or password") for failed login attempts — never revealing whether the email exists.
- **FR-011**: System MUST enforce rate limiting on login attempts to prevent brute-force attacks.
- **FR-012**: System MUST provide a logout function that terminates the user's session.
- **FR-013**: System MUST provide a password recovery flow: request reset by email → receive reset link → set new password.
- **FR-014**: Password reset links MUST expire after a reasonable time period (e.g., 1 hour) and MUST be single-use.
- **FR-015**: System MUST allow Org Admins without active projects to convert their account to a Funder role.
- **FR-016**: System MUST prevent account conversion if the user has projects with status Active, Approved, or Pending Approval.
- **FR-017**: System MUST NOT allow users to self-assign the Super Admin role. Super Admin assignment MUST be an administrative action.
- **FR-018**: System MUST log all authentication events (login, logout, failed login, role changes) for security auditing.
- **FR-019**: System MUST redirect users to the appropriate role-specific dashboard after login.
- **FR-020**: System MUST hide UI elements that a user's role cannot access (in addition to backend enforcement).

### Key Entities

- **User**: A registered account on the platform. Key attributes: id, name, email (unique), password (hashed), company name, company type, role (Org Admin / Funder / Super Admin), email verified flag, created at, updated at.
- **Role**: Determines feature access. Values: ORG_ADMIN, FUNDER, SUPER_ADMIN. A user has exactly one role at a time.
- **Session/Token**: Represents an authenticated user session. Includes: user id, role, expiration. Used for authorization checks on every protected request.
- **Password Reset Token**: A time-limited, single-use token for password recovery. Key attributes: token value, user id, expiration timestamp, used flag.
- **Auth Audit Log**: Records authentication events. Key attributes: event type (login, logout, failed_login, role_change, password_reset), user id, timestamp, IP address, additional context.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 95% of new users can complete registration (form + role selection) in under 2 minutes.
- **SC-002**: Login succeeds within 2 seconds for 99% of requests.
- **SC-003**: 100% of protected endpoints enforce role-based access — no unauthorized access is possible regardless of how the request is made.
- **SC-004**: Failed login attempts are rate-limited after 5 consecutive failures within 15 minutes.
- **SC-005**: Password reset emails are delivered within 60 seconds of the request.
- **SC-006**: All authentication events (login, logout, failure, role change) are logged with sufficient detail for security auditing.
- **SC-007**: Users who convert from Org Admin to Funder can access all funder features immediately after conversion.
- **SC-008**: The registration flow works correctly on desktop, tablet, and mobile devices.

## Scope

### In Scope
- Registration form with name, email, password, company name, company type
- Role selection step (Org Admin / Funder) during registration
- Email and password validation
- User login with email and password
- Session management (login, logout, session/token handling)
- Role-based access control enforcement on all protected endpoints
- UI-level feature hiding based on role
- Password recovery (forgot password → email reset link → set new password)
- Login rate limiting
- Account conversion from Org Admin to Funder
- Authentication event logging
- Super Admin role enforcement (not self-assignable)

### Out of Scope
- Social login / OAuth (e.g., Google, GitHub) — can be added later
- Multi-factor authentication (MFA/2FA) — deferred to a future security hardening phase
- User profile editing beyond role conversion (separate feature)
- Onboarding questionnaire after registration (separate feature — US-16)
- Email verification flow — assumed to exist from Phase 1 or deferred
- Admin panel for user management (Super Admin managing other users)
- Organization entity management (multiple users per org) — deferred

## Assumptions

- Phase 1 already has a basic user model and possibly a simple registration form. This spec extends it with: company fields, role selection, RBAC enforcement, password recovery, and account conversion.
- The existing authentication system uses JWT (access + refresh token pattern) as defined in the project constitution.
- The Super Admin role exists and is seeded via database — it is never selected during registration.
- Rate limiting infrastructure exists or will be added as part of this feature (global rate limiting per constitution Principle V).
- Email sending capability exists on the platform (for password reset emails and optional registration confirmation).
- The platform already has a homepage and basic navigation — this feature adds login/register links and role-based menu items.
