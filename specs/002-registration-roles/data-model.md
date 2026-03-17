# Data Model: Registration & Role-Based Access Control

**Feature**: 002-registration-roles | **Date**: 2026-03-17

## Entities

### User (existing — extended)

The User entity is assumed to exist from Phase 1. This feature extends it with company fields and role.

**Existing fields (from Phase 1):**

| Field | Type | Description |
|-------|------|-------------|
| id | String (CUID) | Primary key |
| name | String | User's full name |
| email | String | Unique email address |
| password_hash | String | Hashed password (bcrypt) |
| is_email_verified | Boolean | Whether email has been verified |
| created_at | DateTime | Account creation timestamp |
| updated_at | DateTime | Last update timestamp |

**New fields (Phase 2):**

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| company_name | String | User's company/organization name | Required for new registrations, nullable for Phase 1 users |
| company_type | String | Type of company/organization | Required for new registrations, nullable for Phase 1 users |
| role | Enum (UserRole) | User's platform role | Required. Default: null (until role selection step is completed) |
| role_selected_at | DateTime | When the user selected their role | Set on role selection, updated on conversion |

**Indexes:**
- `idx_users_email` on `email` (unique) — likely exists from Phase 1
- `idx_users_role` on `role` for role-based queries

---

### PasswordResetToken (new)

Stores hashed password reset tokens with expiration and single-use tracking.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| user_id | String | FK to User | Required |
| token_hash | String | SHA-256 hash of the reset token | Required |
| expires_at | DateTime | Token expiration (1 hour from creation) | Required |
| is_used | Boolean | Whether the token has been used | Default: false |
| created_at | DateTime | When the token was created | Auto-generated |

**Relationships:**
- PasswordResetToken belongs to User (many-to-one)

**Constraints:**
- A user can have multiple reset tokens (previous ones become irrelevant once a new one is created).
- Token is single-use: once `is_used` is true, it cannot be used again.
- Expired tokens (expires_at < now) are invalid regardless of `is_used` status.
- Old tokens should be cleaned up periodically (e.g., delete tokens older than 24 hours).

**Indexes:**
- `idx_password_reset_tokens_user_id` on `user_id`
- `idx_password_reset_tokens_expires_at` on `expires_at` for cleanup queries

---

### RefreshToken (new or extended)

Stores refresh tokens for JWT rotation. May exist from Phase 1 — extend if needed.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| user_id | String | FK to User | Required |
| token_hash | String | SHA-256 hash of the refresh token | Required |
| expires_at | DateTime | Token expiration (7 days from creation) | Required |
| is_revoked | Boolean | Whether the token has been revoked | Default: false |
| created_at | DateTime | When the token was issued | Auto-generated |

**Relationships:**
- RefreshToken belongs to User (many-to-one)

**Constraints:**
- On token refresh: old token is revoked, new token is issued (rotation).
- On logout: current refresh token is revoked.
- On role change: all user's refresh tokens are revoked (forces re-authentication with new role).
- On password reset: all user's refresh tokens are revoked.

**Indexes:**
- `idx_refresh_tokens_user_id` on `user_id`
- `idx_refresh_tokens_token_hash` on `token_hash` for lookup

---

### AuthAuditLog (new)

Records authentication-related events for security auditing.

| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| id | String (CUID) | Primary key | Auto-generated |
| user_id | String | FK to User (nullable for failed logins with unknown email) | Optional |
| event_type | Enum (AuthEventType) | Type of auth event | Required |
| ip_address | String | Client IP address | Required |
| user_agent | String | Client user agent string | Optional |
| metadata | JSON | Additional event context | Optional |
| created_at | DateTime | Event timestamp | Auto-generated |

**Constraints:**
- Never stores passwords or tokens (even hashed) in the log.
- `metadata` may contain: role changed from/to, reason for failure, etc.
- Logs are append-only — never updated or deleted through the application.

**Indexes:**
- `idx_auth_audit_log_user_id` on `user_id`
- `idx_auth_audit_log_event_type` on `event_type`
- `idx_auth_audit_log_created_at` on `created_at` for time-range queries

---

## Enum Definitions

### UserRole

| Value | Description | Self-assignable |
|-------|-------------|-----------------|
| ORG_ADMIN | Organization Administrator — can manage projects | Yes (during registration) |
| FUNDER | Funder — can browse, bookmark, receive notifications | Yes (during registration) |
| SUPER_ADMIN | Platform Administrator — full access | No (database seed / admin action only) |

### AuthEventType

| Value | Description |
|-------|-------------|
| LOGIN_SUCCESS | Successful login |
| LOGIN_FAILED | Failed login attempt (wrong credentials) |
| LOGIN_RATE_LIMITED | Login blocked due to rate limiting |
| LOGOUT | User logged out |
| REGISTER | New account created |
| ROLE_SELECTED | User selected their role during registration |
| ROLE_CONVERTED | User converted from Org Admin to Funder |
| PASSWORD_RESET_REQUESTED | Password reset email sent |
| PASSWORD_RESET_COMPLETED | Password successfully reset |
| TOKEN_REFRESHED | Access token refreshed via refresh token |
| TOKEN_REVOKED | Refresh token revoked (logout, role change, password reset) |

---

## State Transitions

### User Registration Flow

```
[Anonymous Visitor]
        │
        │ register (name, email, password, company)
        v
[Registered — No Role]
        │
        │ select role (ORG_ADMIN or FUNDER)
        v
[Active User with Role]
        │
        │ (optional) convert role
        v
[Active User with New Role]
```

### Account Conversion

```
[Org Admin — no active projects]
        │
        │ convert to Funder
        v
[Funder]
        │
        │ All refresh tokens revoked
        │ New tokens issued with FUNDER role
        v
[Funder — active session with new role]
```

**Conversion Rules:**
- Only ORG_ADMIN → FUNDER is allowed.
- FUNDER → ORG_ADMIN is NOT allowed (would need a new registration or admin action).
- SUPER_ADMIN cannot be reached via conversion.
- Conversion is blocked if user has projects with status: ACTIVE, APPROVED, or PENDING_APPROVAL.
