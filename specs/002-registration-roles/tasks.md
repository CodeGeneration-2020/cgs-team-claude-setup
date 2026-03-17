# Tasks: Registration & Role-Based Access Control

**Input**: Design documents from `/specs/002-registration-roles/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: TDD — constitution requires 80% coverage.

**Organization**: Tasks grouped by user story. US1+US2+US4 are grouped as the P1 MVP (registration + role selection + login are inseparable). US3 (RBAC) is a separate P1 phase since it's a cross-cutting concern applied after auth works.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Maps to user story from spec.md

## Path Conventions

- **Backend**: `apps/<backend-app>/src/`
- **Frontend**: `apps/<web-app>/src/features/`
- **Shared types**: `packages/shared-types/src/`
- **API client**: `packages/api-client/src/`
- **E2E tests**: `tests/e2e/auth/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Shared types, enums, and API client stubs

- [ ] T001 Add UserRole enum (ORG_ADMIN, FUNDER, SUPER_ADMIN) in packages/shared-types/src/auth/role.ts
- [ ] T002 [P] Add auth request/response types in packages/shared-types/src/auth/login.ts, register.ts, auth-response.ts, index.ts
- [ ] T003 [P] Add user profile type in packages/shared-types/src/user/user-profile.ts, index.ts
- [ ] T004 [P] Add AuthEventType enum in packages/shared-types/src/auth/auth-event-type.ts
- [ ] T005 Re-export all new types from packages/shared-types/src/index.ts barrel

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Database migrations, guards, decorators — MUST complete before any user story

**CRITICAL**: No user story work can begin until this phase is complete

- [ ] T006 Create Prisma migration to extend users table: add company_name (String, nullable), company_type (String, nullable), role (UserRole enum, nullable), role_selected_at (DateTime, nullable). Add index idx_users_role
- [ ] T007 [P] Create Prisma migration for password_reset_tokens table: id, user_id (FK), token_hash, expires_at, is_used (default false), created_at. Add indexes idx_password_reset_tokens_user_id, idx_password_reset_tokens_expires_at
- [ ] T008 [P] Create Prisma migration for refresh_tokens table (or extend if exists): id, user_id (FK), token_hash, expires_at, is_revoked (default false), created_at. Add indexes idx_refresh_tokens_user_id, idx_refresh_tokens_token_hash
- [ ] T009 [P] Create Prisma migration for auth_audit_log table: id, user_id (FK nullable), event_type (AuthEventType enum), ip_address, user_agent (nullable), metadata (JSON nullable), created_at. Add indexes on user_id, event_type, created_at
- [ ] T010 Create @Public() decorator in apps/<backend-app>/src/common/decorators/public.decorator.ts — sets metadata to skip auth guard
- [ ] T011 [P] Create @Roles() decorator in apps/<backend-app>/src/common/decorators/roles.decorator.ts — sets required roles metadata on route
- [ ] T012 [P] Create @CurrentUser() decorator in apps/<backend-app>/src/common/decorators/current-user.decorator.ts — extracts user from request
- [ ] T013 Create global AuthGuard in apps/<backend-app>/src/common/guards/auth.guard.ts — validates JWT, skips routes marked @Public(), attaches user to request
- [ ] T014 Create RolesGuard in apps/<backend-app>/src/common/guards/roles.guard.ts — reads @Roles() metadata, checks request.user.role against required roles, returns 403 if unauthorized
- [ ] T015 Register AuthGuard as global guard and RolesGuard as global guard in the app module
- [ ] T016 [P] Create auth audit interceptor in apps/<backend-app>/src/common/interceptors/auth-audit.interceptor.ts — logs auth events to auth_audit_log table
- [ ] T017 Create seed script for Super Admin account in prisma/seed.ts (or extend existing seed)
- [ ] T018 Add auth API client methods in packages/api-client/src/auth/auth.client.ts — register(), login(), refreshToken(), logout(), selectRole(), convertRole(), forgotPassword(), resetPassword(), getMe()
- [ ] T019 Write unit tests for AuthGuard in apps/<backend-app>/src/common/guards/__tests__/auth.guard.spec.ts
- [ ] T020 [P] Write unit tests for RolesGuard in apps/<backend-app>/src/common/guards/__tests__/roles.guard.spec.ts

**Checkpoint**: Foundation ready — database schema, guards, decorators, API client all in place

---

## Phase 3: US1 + US2 + US4 — Registration + Role Selection + Login (Priority: P1) MVP

**Goal**: User can register with all required fields, select a role (Org Admin / Funder), log in with email/password, and be redirected to the appropriate dashboard. Includes logout and token refresh.

**Independent Test**: Navigate to /register → fill form → submit → select role → verify redirect. Navigate to /login → enter credentials → verify authentication and role-appropriate redirect.

### Backend — Auth Module

- [ ] T021 [US1] Create auth module scaffold: apps/<backend-app>/src/modules/auth/auth.module.ts, auth.controller.ts, auth.service.ts, dto/, entities/, index.ts
- [ ] T022 [P] [US1] Create RegisterDto in apps/<backend-app>/src/modules/auth/dto/register.dto.ts — validates name, email, password (min 8, uppercase, lowercase, number, special char), companyName, companyType
- [ ] T023 [P] [US4] Create LoginDto in apps/<backend-app>/src/modules/auth/dto/login.dto.ts — validates email, password
- [ ] T024 [P] [US1] Create AuthResponseDto in apps/<backend-app>/src/modules/auth/dto/auth-response.dto.ts — accessToken, refreshToken, expiresIn, user object, needsRoleSelection flag
- [ ] T025 [US1] Implement AuthService.register() in apps/<backend-app>/src/modules/auth/auth.service.ts — hash password with bcrypt (cost 12), create user, return userId, log REGISTER event
- [ ] T026 [US4] Implement AuthService.login() — validate credentials, check rate limit, generate JWT access + refresh tokens with role in payload, store refresh token hash, log LOGIN_SUCCESS or LOGIN_FAILED event
- [ ] T027 [US4] Implement AuthService.refreshToken() — validate refresh token hash, check expiry/revoked, rotate: revoke old + issue new, log TOKEN_REFRESHED
- [ ] T028 [US4] Implement AuthService.logout() — revoke refresh token, log LOGOUT event
- [ ] T029 [US1] Implement POST /v1/auth/register in AuthController — @Public(), delegates to service, returns 201 with userId or 409 for duplicate email
- [ ] T030 [US4] Implement POST /v1/auth/login in AuthController — @Public(), delegates to service, returns tokens + user or 401/429
- [ ] T031 [US4] Implement POST /v1/auth/refresh in AuthController — @Public(), delegates to service
- [ ] T032 [US4] Implement POST /v1/auth/logout in AuthController — requires auth, delegates to service
- [ ] T033 [US4] Add rate limiting configuration for login endpoint — 5 attempts per 15 minutes per IP

### Backend — Users Module (Role Selection)

- [ ] T034 [US2] Create users module scaffold: apps/<backend-app>/src/modules/users/users.module.ts, users.controller.ts, users.service.ts, dto/, index.ts
- [ ] T035 [US2] Create SelectRoleDto in apps/<backend-app>/src/modules/users/dto/select-role.dto.ts — validates role is ORG_ADMIN or FUNDER (rejects SUPER_ADMIN)
- [ ] T036 [US2] Implement UsersService.selectRole() in apps/<backend-app>/src/modules/users/users.service.ts — set role on user, invalidate old tokens, issue new tokens with role, log ROLE_SELECTED event
- [ ] T037 [US2] Implement POST /v1/users/select-role in UsersController — requires auth + role must be null, returns new tokens with role
- [ ] T038 Implement GET /v1/users/me in UsersController — returns current user profile with canConvertToFunder flag

### Backend Tests

- [ ] T039 [US1] Write unit tests for AuthService.register() in apps/<backend-app>/src/modules/auth/__tests__/auth.service.spec.ts — valid registration, duplicate email, validation errors, password hashing
- [ ] T040 [P] [US4] Write unit tests for AuthService.login() — valid login, wrong password, unknown email, rate limiting, token generation
- [ ] T041 [P] [US2] Write unit tests for UsersService.selectRole() in apps/<backend-app>/src/modules/users/__tests__/users.service.spec.ts — valid selection, SUPER_ADMIN rejection, already-selected
- [ ] T042 [US1] Write integration tests for auth endpoints in apps/<backend-app>/src/modules/auth/__tests__/auth.integration.spec.ts — register + login + refresh + logout flow
- [ ] T043 [P] [US2] Write integration test for role selection in apps/<backend-app>/src/modules/users/__tests__/users.integration.spec.ts

### Frontend — Auth Feature Module

- [ ] T044 [US1] Create auth feature module scaffold: apps/<web-app>/src/features/auth/ with ui/pages/, ui/components/, state/, services/, hooks/, domain/, index.ts
- [ ] T045 [US4] Create useAuthStore (Zustand) in apps/<web-app>/src/features/auth/state/useAuthStore.ts — manages: isAuthenticated, user, role, tokens, login(), logout(), setRole()
- [ ] T046 [US1] Create RegisterForm component in apps/<web-app>/src/features/auth/ui/components/RegisterForm.tsx — fields: name, email, password, companyName, companyType. Client-side validation matching backend rules
- [ ] T047 [US1] Create RegisterPage in apps/<web-app>/src/features/auth/ui/pages/RegisterPage.tsx — renders RegisterForm, handles submit via API client, redirects to role selection on success
- [ ] T048 [US2] Create RoleSelector component in apps/<web-app>/src/features/auth/ui/components/RoleSelector.tsx — two options: Org Admin / Funder, confirm button disabled until selection
- [ ] T049 [US2] Create RoleSelectionPage in apps/<web-app>/src/features/auth/ui/pages/RoleSelectionPage.tsx — renders RoleSelector, calls selectRole API, updates auth store, redirects to dashboard
- [ ] T050 [US4] Create LoginForm component in apps/<web-app>/src/features/auth/ui/components/LoginForm.tsx — email + password fields, "Forgot password" link, submit
- [ ] T051 [US4] Create LoginPage in apps/<web-app>/src/features/auth/ui/pages/LoginPage.tsx — renders LoginForm, handles auth, checks needsRoleSelection, redirects appropriately
- [ ] T052 [US4] Create useAuth hook in apps/<web-app>/src/features/auth/hooks/useAuth.ts — wraps TanStack Query mutations for login, register, refresh, logout
- [ ] T053 [US4] Implement token refresh logic — auto-refresh access token before expiry using refresh token, update auth store
- [ ] T054 Add login/register links to global navigation and implement role-aware menu rendering (hide items user can't access)

### Frontend Tests

- [ ] T055 [US1] Write component tests for RegisterForm in apps/<web-app>/src/features/auth/__tests__/RegisterForm.spec.tsx
- [ ] T056 [P] [US4] Write component tests for LoginForm in apps/<web-app>/src/features/auth/__tests__/LoginForm.spec.tsx
- [ ] T057 [P] [US2] Write component tests for RoleSelector in apps/<web-app>/src/features/auth/__tests__/RoleSelector.spec.tsx

**Checkpoint**: US1+US2+US4 complete — user can register, select role, log in, see role-appropriate UI

---

## Phase 4: US3 — Role-Based Feature Access (Priority: P1)

**Goal**: All protected endpoints enforce RBAC. Unauthorized access returns 403. Frontend hides inaccessible features and protects routes.

**Independent Test**: Log in as each role → verify correct features accessible → attempt unauthorized access via URL/API → verify 403.

### Backend — RBAC Enforcement

- [ ] T058 [US3] Apply @Roles() decorator to all existing protected endpoints across the backend — project CRUD (ORG_ADMIN), trending admin (SUPER_ADMIN), bookmarks (FUNDER when implemented)
- [ ] T059 [US3] Apply @Public() decorator to all public endpoints — homepage search, trending, project detail, auth endpoints
- [ ] T060 [US3] Write RBAC integration tests in apps/<backend-app>/src/common/guards/__tests__/rbac.integration.spec.ts — test each role against each endpoint category, verify 401 for unauthenticated, 403 for wrong role

### Frontend — Route Protection

- [ ] T061 [US3] Create useRoleGuard hook in apps/<web-app>/src/features/auth/hooks/useRoleGuard.ts — checks auth state and role, returns redirect path if unauthorized
- [ ] T062 [US3] Create ProtectedRoute wrapper component in apps/<web-app>/src/features/auth/ui/components/ProtectedRoute.tsx — wraps routes requiring auth, redirects to login or shows 403 based on role
- [ ] T063 [US3] Wrap all protected frontend routes with ProtectedRoute — project management routes (ORG_ADMIN), bookmark routes (FUNDER), admin routes (SUPER_ADMIN)
- [ ] T064 [US3] Create ForbiddenPage (403) in apps/<web-app>/src/features/auth/ui/pages/ForbiddenPage.tsx — "You don't have permission" message with link to dashboard
- [ ] T065 [US3] Write component test for ProtectedRoute in apps/<web-app>/src/features/auth/__tests__/ProtectedRoute.spec.tsx — test redirect for unauthenticated, 403 for wrong role, pass-through for correct role

**Checkpoint**: US3 complete — RBAC enforced on all endpoints and routes, unauthorized access blocked

---

## Phase 5: US5 — Password Recovery (Priority: P2)

**Goal**: User can request a password reset via email, receive a time-limited link, and set a new password.

**Independent Test**: Click "Forgot password" → enter email → check email for reset link → click link → set new password → log in with new password.

### Backend

- [ ] T066 [US5] Create ForgotPasswordDto in apps/<backend-app>/src/modules/auth/dto/forgot-password.dto.ts — validates email
- [ ] T067 [P] [US5] Create ResetPasswordDto in apps/<backend-app>/src/modules/auth/dto/reset-password.dto.ts — validates token (required) and newPassword (same rules as registration)
- [ ] T068 [US5] Create PasswordResetToken entity in apps/<backend-app>/src/modules/auth/entities/password-reset-token.entity.ts — maps to password_reset_tokens table
- [ ] T069 [US5] Implement AuthService.forgotPassword() — generate 32-byte random token, hash with SHA-256, store in DB with 1-hour expiry, send email with reset link. Always return success (no email enumeration)
- [ ] T070 [US5] Implement AuthService.resetPassword() — verify token hash, check expiry + is_used, update password, mark token used, revoke all user's refresh tokens, log PASSWORD_RESET_COMPLETED
- [ ] T071 [US5] Implement POST /v1/auth/forgot-password in AuthController — @Public(), returns same response regardless of email existence
- [ ] T072 [US5] Implement POST /v1/auth/reset-password in AuthController — @Public(), validates token + new password
- [ ] T073 [US5] Write unit tests for forgot/reset password in apps/<backend-app>/src/modules/auth/__tests__/password-reset.spec.ts — valid flow, expired token, used token, unknown email, weak password

### Frontend

- [ ] T074 [US5] Create ForgotPasswordForm in apps/<web-app>/src/features/auth/ui/components/ForgotPasswordForm.tsx — email input, submit, success message
- [ ] T075 [US5] Create ForgotPasswordPage in apps/<web-app>/src/features/auth/ui/pages/ForgotPasswordPage.tsx — renders form, linked from login page
- [ ] T076 [US5] Create ResetPasswordForm in apps/<web-app>/src/features/auth/ui/components/ResetPasswordForm.tsx — new password + confirm, validates matching + strength
- [ ] T077 [US5] Create ResetPasswordPage in apps/<web-app>/src/features/auth/ui/pages/ResetPasswordPage.tsx — reads token from URL, renders form, redirects to login on success, shows expired message if invalid
- [ ] T078 [US5] Write component tests for ForgotPasswordForm and ResetPasswordForm in apps/<web-app>/src/features/auth/__tests__/

**Checkpoint**: US5 complete — password recovery works end-to-end

---

## Phase 6: US6 — Account Conversion (Priority: P3)

**Goal**: Org Admin without active projects can convert to Funder. Role changes immediately, tokens refreshed, project management features hidden.

**Independent Test**: Log in as Org Admin (no active projects) → account settings → convert → verify funder features accessible, project management hidden.

### Backend

- [ ] T079 [US6] Create ConvertRoleDto in apps/<backend-app>/src/modules/users/dto/convert-role.dto.ts — validates targetRole is FUNDER
- [ ] T080 [US6] Implement UsersService.convertRole() — check current role is ORG_ADMIN, check no active/approved/pending projects, update role to FUNDER, revoke all refresh tokens, issue new tokens, log ROLE_CONVERTED
- [ ] T081 [US6] Implement POST /v1/users/convert-role in UsersController — @Roles(UserRole.ORG_ADMIN), returns new tokens with FUNDER role or error
- [ ] T082 [US6] Write unit tests for convertRole in apps/<backend-app>/src/modules/users/__tests__/convert-role.spec.ts — valid conversion, has active projects (blocked), wrong role (blocked)

### Frontend

- [ ] T083 [US6] Add "Convert to Funder" option in account settings — visible only for ORG_ADMIN when canConvertToFunder is true
- [ ] T084 [US6] Create conversion confirmation dialog — warns about losing project management access, requires explicit confirmation
- [ ] T085 [US6] On conversion success — update auth store with new role + tokens, redirect to funder dashboard, navigation updates immediately
- [ ] T086 [US6] Write component test for account conversion flow in apps/<web-app>/src/features/auth/__tests__/ConvertRole.spec.tsx

**Checkpoint**: US6 complete — account conversion works, role changes reflected immediately

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: E2E tests, responsive design, edge cases

- [ ] T087 [P] Implement form data preservation on session expiry — client-side storage of registration form state
- [ ] T088 [P] Add responsive layout for all auth pages — register, login, role selection, password recovery (mobile 375px, tablet 768px, desktop 1440px)
- [ ] T089 [P] Write E2E test for registration flow in tests/e2e/auth/registration.spec.ts — full flow from /register to role selection to dashboard
- [ ] T090 [P] Write E2E test for login flow in tests/e2e/auth/login.spec.ts — valid login, invalid credentials, rate limiting, logout
- [ ] T091 [P] Write E2E test for RBAC in tests/e2e/auth/rbac.spec.ts — verify each role sees correct features, 403 for unauthorized
- [ ] T092 [P] Write E2E test for password recovery in tests/e2e/auth/password-recovery.spec.ts
- [ ] T093 Add cleanup job for expired password reset tokens (delete tokens older than 24 hours)
- [ ] T094 Run quickstart.md validation — verify all setup steps, commands, and seed script work

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies — start immediately
- **Phase 2 (Foundation)**: Depends on Phase 1 — BLOCKS all user stories
- **Phase 3 (US1+US2+US4)**: Depends on Phase 2 — MVP registration + login
- **Phase 4 (US3)**: Depends on Phase 3 — applies RBAC to working auth system
- **Phase 5 (US5)**: Depends on Phase 3 (extends auth module). Can run in parallel with Phase 4
- **Phase 6 (US6)**: Depends on Phase 3 + Phase 4 (needs working RBAC)
- **Phase 7 (Polish)**: Depends on all desired user stories being complete

### User Story Dependencies

```
Phase 1 (Setup)
    │
Phase 2 (Foundation: guards, decorators, DB schema)
    │
Phase 3 (US1+US2+US4: Register + Role + Login) ─── MVP
    │
    ├──────────────┐
    │              │
Phase 4          Phase 5
US3 RBAC         US5 Password Recovery
    │              (parallel with Phase 4)
    │
Phase 6
US6 Account Conversion
    │
Phase 7 (Polish + E2E)
```

### Within Each Phase

- DTOs before services
- Services before controllers
- Backend before frontend integration
- Unit tests alongside implementation
- Integration tests after endpoints are complete

### Parallel Opportunities

- **Phase 1**: T001-T005 all types can be created in parallel
- **Phase 2**: T006-T009 migrations in parallel, T010-T012 decorators in parallel, T019-T020 guard tests in parallel
- **Phase 3**: Backend DTOs (T022-T024) in parallel, service tests (T039-T041) in parallel, frontend tests (T055-T057) in parallel
- **Phase 4+5**: Can run in parallel (RBAC and password recovery are independent)
- **Phase 7**: All E2E tests (T089-T092) in parallel

---

## Parallel Example: Phase 3

```bash
# After T021 (module scaffold), launch DTOs in parallel:
Task T022: "Create RegisterDto"
Task T023: "Create LoginDto"
Task T024: "Create AuthResponseDto"

# After T025-T028 (services), launch tests in parallel:
Task T039: "Unit tests for register"
Task T040: "Unit tests for login"
Task T041: "Unit tests for selectRole"

# Frontend components in parallel (after auth store T045):
Task T046: "RegisterForm"
Task T050: "LoginForm"
Task T048: "RoleSelector"
```

---

## Implementation Strategy

### MVP First (Phase 1 + 2 + 3 = US1+US2+US4)

1. Complete Phase 1: Shared types
2. Complete Phase 2: DB schema, guards, decorators, API client
3. Complete Phase 3: Register + role selection + login
4. **STOP and VALIDATE**: User can register, pick a role, log in, see role-appropriate UI
5. Deploy/demo MVP

### Incremental Delivery

1. **Phase 1+2+3** → Registration + login works (MVP)
2. **Phase 4** → RBAC enforced everywhere → Deploy
3. **Phase 5** → Password recovery → Deploy
4. **Phase 6** → Account conversion → Deploy
5. **Phase 7** → E2E tests, polish → Final release

### Parallel Team Strategy

With 2 developers after Phase 2:
- **Dev A**: Phase 3 (registration + login) → Phase 4 (RBAC)
- **Dev B**: Phase 5 (password recovery, starts after Phase 3 backend is done)
- **Together**: Phase 6 (account conversion) → Phase 7 (polish)

---

## Notes

- [P] tasks = different files, no dependencies on incomplete tasks
- [Story] label maps to spec.md user stories
- US1+US2+US4 grouped because registration, role selection, and login are inseparable for a working auth system
- US3 (RBAC) is a separate phase because it applies guards across the entire app — needs working auth first
- Constitution requires: JWT access+refresh, bcrypt hashing, class-validator DTOs, guards not service-level RBAC, structured Pino logging, 80% test coverage
- Super Admin is NEVER self-assignable — seeded via database only
- All auth error messages must be generic to prevent user enumeration (FR-010)
