# Research: Registration & Role-Based Access Control

**Feature**: 002-registration-roles | **Date**: 2026-03-17

## R1: JWT Token Strategy

**Decision**: Use access token + refresh token pattern with role encoded in the access token payload.

**Rationale**: Constitution Principle V mandates JWT with access + refresh token rotation. The user's role MUST be included in the access token so that RBAC guards can authorize requests without a database lookup on every request. Refresh tokens are stored server-side (database) for revocation support.

**Alternatives Considered**:
- **Session-based auth**: Violates constitution requirement for JWT. Also harder to scale horizontally (requires sticky sessions or shared session store).
- **Role lookup on every request**: Accurate but adds a DB query to every protected endpoint. Include role in JWT and re-issue token on role change instead.

**Implementation Notes**:
- Access token: short-lived (15 minutes), contains user id + role + email.
- Refresh token: long-lived (7 days), stored in database, rotated on each use.
- On role change (account conversion), invalidate existing tokens and issue new ones with updated role.

---

## R2: RBAC Guard Architecture

**Decision**: Implement RBAC using a custom `@Roles()` decorator + `RolesGuard` that reads the role from the JWT payload and checks against the required roles for the endpoint.

**Rationale**: Constitution Principle V states "Authorisation: Role-Based Access Control (RBAC) via NestJS role guards using roles defined in `/packages/shared-types`; role checks MUST never live in service logic." A decorator-based approach keeps authorization declarative and separate from business logic.

**Alternatives Considered**:
- **Role checks in service methods**: Violates constitution — role checks must be in guards, not services.
- **Middleware-based RBAC**: Less granular than guard-based — can't apply per-route role requirements.
- **CASL/ability-based**: More flexible but over-engineered for 3 simple roles. Can migrate later if needed.

**Implementation Notes**:
- Global Auth Guard applied to all routes by default (constitution: "all routes are protected by default").
- `@Public()` decorator opts specific routes out of authentication (homepage, search, trending, project detail, auth endpoints).
- `@Roles(UserRole.ORG_ADMIN)` decorator specifies required role(s) per endpoint.
- `RolesGuard` reads `request.user.role` from JWT and checks against decorator metadata.

---

## R3: Password Hashing

**Decision**: Use bcrypt for password hashing with a cost factor of 12.

**Rationale**: bcrypt is the industry standard for password hashing. Cost factor 12 provides strong security while keeping login response time under 2 seconds (SC-002). Constitution requires passwords to be validated via class-validator DTOs.

**Alternatives Considered**:
- **Argon2**: Newer and configurable, but bcrypt is universally supported and meets security requirements.
- **scrypt**: Good alternative but less common in the Node.js ecosystem.

---

## R4: Rate Limiting Strategy for Login

**Decision**: Apply per-IP rate limiting on the login endpoint: 5 attempts per 15 minutes. Use the global rate limiting infrastructure with a per-route override for stricter auth limits.

**Rationale**: SC-004 requires rate limiting after 5 failures in 15 minutes. Constitution Principle V mandates "Rate limiting MUST be applied globally with configurable per-route overrides." The login endpoint gets a stricter limit than the global default.

**Alternatives Considered**:
- **Per-account rate limiting**: More accurate but requires tracking by email, which leaks information about which emails exist.
- **CAPTCHA after N failures**: Good UX but adds complexity. Can layer on top of rate limiting later.

---

## R5: Password Reset Token Design

**Decision**: Generate a cryptographically random token, store it hashed in the database with a 1-hour expiration and single-use flag.

**Rationale**: FR-014 requires reset links to expire and be single-use. Storing the token hashed (not plaintext) prevents exposure if the database is compromised. 1-hour expiration balances security with user convenience.

**Implementation Notes**:
- Generate 32-byte random token, send as URL parameter in email.
- Store SHA-256 hash of token in `password_reset_tokens` table.
- On use: verify hash match, check expiration, mark as used, update password, invalidate all existing refresh tokens for the user.

---

## R6: Phase 1 Extension Strategy

**Decision**: Extend existing user model and auth module rather than rebuilding. Add new fields (company_name, company_type, role) via Prisma migration. Add new endpoints alongside existing ones.

**Rationale**: The user confirmed Phase 1 already has basic registration. Extending is less risky than replacing — existing users and data are preserved. New fields are nullable initially to avoid breaking existing accounts, then backfilled.

**Alternatives Considered**:
- **Full rebuild**: Cleaner but risks breaking existing users and losing Phase 1 data.
- **Separate auth service**: Over-engineered for the current scale. A single module extension is sufficient.

---

## R7: Frontend Route Protection

**Decision**: Implement a `useRoleGuard` hook and `ProtectedRoute` wrapper component that checks authentication state and role from the Zustand auth store.

**Rationale**: FR-020 requires UI-level feature hiding in addition to backend enforcement. A route-level guard prevents unauthorized users from even seeing protected pages. Combined with conditional menu rendering based on role.

**Implementation Notes**:
- `useAuthStore` holds: isAuthenticated, user, role, tokens.
- `ProtectedRoute` checks auth state — redirects to login if unauthenticated, shows 403 page if wrong role.
- Navigation menu conditionally renders items based on `user.role`.
- Backend enforcement is the source of truth — frontend guards are a UX improvement only.
