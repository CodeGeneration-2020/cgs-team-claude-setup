# Implementation Plan: Registration & Role-Based Access Control

**Branch**: `002-registration-roles` | **Date**: 2026-03-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/002-registration-roles/spec.md`

## Summary

Extend the existing Phase 1 registration system with company fields, a post-registration role selection step (Org Admin / Funder), JWT-based authentication with role-aware tokens, comprehensive RBAC enforcement via NestJS guards on all protected endpoints, password recovery flow, account conversion (Org Admin → Funder), and authentication audit logging. The frontend adds registration, login, role selection, and password recovery pages, plus role-aware navigation that hides unauthorized features.

## Technical Context

**Language/Version**: TypeScript (strict mode)
**Primary Dependencies**: NestJS (backend), React.js + Tailwind CSS (frontend), Zustand + TanStack Query (state/data), Prisma ORM
**Storage**: PostgreSQL (users, roles, password reset tokens, audit logs)
**Testing**: Jest (unit + integration), Playwright (E2E), React Testing Library (components)
**Target Platform**: Web application (desktop + tablet + mobile responsive)
**Project Type**: Web service (monorepo: backend API + frontend SPA)
**Performance Goals**: Login within 2 seconds for 99% of requests, registration form completion < 2 minutes
**Constraints**: JWT access + refresh token pattern (per constitution), rate limiting on auth endpoints, no secrets in source code, generic error messages to prevent user enumeration
**Scale/Scope**: Thousands of users, 3 roles (ORG_ADMIN, FUNDER, SUPER_ADMIN)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Monorepo-First Architecture | PASS | Feature extends existing backend + frontend apps in monorepo |
| II. Clean Architecture & SOLID | PASS | Auth logic in service layer, controllers delegate only, guards handle cross-cutting auth/RBAC |
| III. Modular Architecture | PASS | Backend: extends existing `auth` module or creates new `auth` + `users` modules. Frontend: `auth` feature module |
| IV. Strict Type Safety | PASS | User, Role, Auth DTOs defined in `/packages/shared-types` |
| V. Security by Design | PASS | JWT with refresh rotation, global Auth Guard, RBAC via role guards, rate limiting on login, helmet middleware, class-validator DTOs, no secrets in code |
| VI. Testing Discipline | PASS | Unit tests for auth service, integration tests for auth endpoints, E2E for registration/login flows |
| VII. Independent Deployability | PASS | No new apps, extends existing backend + frontend |
| VIII. Observability-First | PASS | Auth events logged via structured Pino logging, all login/logout/failure/role-change events captured |
| IX. Shared-Before-Custom | PASS | Auth types in shared-types, auth client methods in api-client |
| X. Design Token Management | PASS | Auth pages use design tokens for styling |

**Gate Result: PASS** — No violations. Proceeding to Phase 0.

## Project Structure

### Documentation (this feature)

```text
specs/002-registration-roles/
├── spec.md
├── plan.md              # This file
├── research.md          # Phase 0: research decisions
├── data-model.md        # Phase 1: entity schemas
├── quickstart.md        # Phase 1: dev environment setup
├── contracts/           # Phase 1: API contracts
│   ├── auth.md
│   └── users.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit.tasks)
```

### Source Code (repository root)

```text
# Backend (NestJS) — extends/creates auth module
apps/<backend-app>/src/
├── common/
│   ├── guards/
│   │   ├── auth.guard.ts              # Global JWT auth guard (may exist from Phase 1)
│   │   └── roles.guard.ts            # RBAC role guard (new)
│   ├── decorators/
│   │   ├── public.decorator.ts        # Mark routes as public (may exist)
│   │   ├── roles.decorator.ts         # @Roles(Role.ORG_ADMIN) decorator (new)
│   │   └── current-user.decorator.ts  # Extract current user from request (may exist)
│   └── interceptors/
│       └── auth-audit.interceptor.ts  # Log auth events (new)
├── modules/
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts         # login, register, refresh, logout, forgot-password, reset-password
│   │   ├── auth.service.ts            # JWT generation, validation, password hashing
│   │   ├── dto/
│   │   │   ├── register.dto.ts
│   │   │   ├── login.dto.ts
│   │   │   ├── select-role.dto.ts
│   │   │   ├── forgot-password.dto.ts
│   │   │   ├── reset-password.dto.ts
│   │   │   └── auth-response.dto.ts
│   │   ├── entities/
│   │   │   └── password-reset-token.entity.ts
│   │   ├── __tests__/
│   │   └── index.ts
│   └── users/
│       ├── users.module.ts
│       ├── users.controller.ts        # role selection, account conversion, profile
│       ├── users.service.ts
│       ├── dto/
│       │   ├── select-role.dto.ts
│       │   └── convert-role.dto.ts
│       ├── __tests__/
│       └── index.ts

# Frontend (React)
apps/<web-app>/src/features/
├── auth/
│   ├── ui/
│   │   ├── pages/
│   │   │   ├── LoginPage.tsx
│   │   │   ├── RegisterPage.tsx
│   │   │   ├── RoleSelectionPage.tsx
│   │   │   ├── ForgotPasswordPage.tsx
│   │   │   └── ResetPasswordPage.tsx
│   │   └── components/
│   │       ├── LoginForm.tsx
│   │       ├── RegisterForm.tsx
│   │       ├── RoleSelector.tsx
│   │       ├── ForgotPasswordForm.tsx
│   │       └── ResetPasswordForm.tsx
│   ├── state/
│   │   └── useAuthStore.ts            # Zustand store: user, role, tokens, isAuthenticated
│   ├── services/
│   │   └── auth.service.ts
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useRoleGuard.ts            # Frontend route protection
│   │   └── useCurrentUser.ts
│   ├── domain/
│   │   └── types.ts
│   ├── __tests__/
│   └── index.ts

# Shared packages
packages/shared-types/src/
├── auth/
│   ├── login.ts
│   ├── register.ts
│   ├── auth-response.ts
│   ├── role.ts                        # UserRole enum: ORG_ADMIN, FUNDER, SUPER_ADMIN
│   └── index.ts
├── user/
│   ├── user-profile.ts
│   └── index.ts

packages/api-client/src/
├── auth/
│   ├── auth.client.ts                 # login, register, selectRole, refreshToken, logout, forgotPassword, resetPassword
│   └── index.ts

# E2E tests
tests/e2e/auth/
├── registration.spec.ts
├── login.spec.ts
├── role-selection.spec.ts
├── password-recovery.spec.ts
└── rbac.spec.ts
```

**Structure Decision**: Extends the existing monorepo. Backend adds/extends `auth` and `users` modules with RBAC guards in `common/guards/`. Frontend adds `auth` feature module with pages for all auth flows. Shared types and API client extended with auth contracts. Follows Constitution Principles I, III, V.

## Complexity Tracking

No violations — no entries needed.
