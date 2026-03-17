# Quickstart: Registration & Role-Based Access Control

**Feature**: 002-registration-roles | **Date**: 2026-03-17

## Prerequisites

- Node.js (LTS)
- pnpm
- PostgreSQL instance
- Email service configured (for password reset emails)
- Environment variables configured (see below)

## Environment Variables

```bash
# Database
APP_DATABASE_URL=postgresql://user:password@localhost:5432/cfc_dev

# JWT
APP_JWT_SECRET=<your-jwt-secret-min-32-chars>
APP_JWT_ACCESS_EXPIRY=900         # 15 minutes in seconds
APP_JWT_REFRESH_EXPIRY=604800     # 7 days in seconds

# Email
APP_EMAIL_SERVICE_URL=<email-service-url>
APP_EMAIL_FROM=noreply@cfcplatform.org

# Rate Limiting
APP_LOGIN_RATE_LIMIT_MAX=5        # max attempts
APP_LOGIN_RATE_LIMIT_WINDOW=900   # 15 minutes in seconds

# Password Reset
APP_PASSWORD_RESET_EXPIRY=3600    # 1 hour in seconds
APP_PASSWORD_RESET_URL=https://cfcplatform.org/reset-password  # frontend URL for reset link

# General
APP_PORT=3000
APP_NODE_ENV=development
```

## Setup

```bash
# Install dependencies
pnpm install

# Generate Prisma client (includes new models)
pnpm --filter <backend-app> prisma generate

# Run database migrations (adds company fields, role, password_reset_tokens, refresh_tokens, auth_audit_log)
pnpm --filter <backend-app> prisma migrate dev

# Seed Super Admin account
pnpm --filter <backend-app> prisma db seed
```

## Development

```bash
# Start all apps
pnpm dev

# Start backend only
pnpm --filter <backend-app> dev

# Start frontend only
pnpm --filter <web-app> dev
```

## Testing

```bash
# Run all tests
pnpm test

# Backend auth module tests
pnpm --filter <backend-app> test -- --testPathPattern=auth

# Backend users module tests
pnpm --filter <backend-app> test -- --testPathPattern=users

# Frontend auth feature tests
pnpm --filter <web-app> test -- --testPathPattern=auth

# E2E auth tests
pnpm --filter <web-app> test:e2e -- --testPathPattern=auth

# Run with coverage
pnpm test -- --coverage
```

## Key Development Notes

### Backend Modules
1. **Auth Module** (`src/modules/auth/`) — register, login, refresh, logout, forgot-password, reset-password
2. **Users Module** (`src/modules/users/`) — role selection, account conversion, user profile
3. **Common Guards** (`src/common/guards/`) — AuthGuard (global), RolesGuard (per-endpoint)
4. **Common Decorators** (`src/common/decorators/`) — @Public(), @Roles(), @CurrentUser()

### Frontend Feature Module
1. **Auth** (`src/features/auth/`) — login, register, role selection, password recovery pages + Zustand auth store

### Shared Packages
1. **shared-types** — UserRole enum, auth DTOs, user profile types
2. **api-client** — auth API client (login, register, selectRole, refresh, logout, forgotPassword, resetPassword)

### Database Changes
1. Add `company_name`, `company_type`, `role`, `role_selected_at` columns to users table
2. Create `password_reset_tokens` table
3. Create or extend `refresh_tokens` table
4. Create `auth_audit_log` table
5. Seed Super Admin account

### API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /v1/auth/register | Public | Create new account |
| POST | /v1/auth/login | Public | Authenticate user |
| POST | /v1/auth/refresh | Public | Refresh access token |
| POST | /v1/auth/logout | Required | End session |
| POST | /v1/auth/forgot-password | Public | Request password reset |
| POST | /v1/auth/reset-password | Public | Reset password with token |
| POST | /v1/users/select-role | Required | Set role after registration |
| POST | /v1/users/convert-role | ORG_ADMIN | Convert to Funder |
| GET | /v1/users/me | Required | Get current user profile |

### RBAC Role Matrix

| Feature | Visitor | Funder | Org Admin | Super Admin |
|---------|---------|--------|-----------|-------------|
| Homepage / Search | Yes | Yes | Yes | Yes |
| Project Detail | Yes | Yes | Yes | Yes |
| Bookmarks | No | Yes | No | Yes |
| Project CRUD | No | No | Yes | Yes |
| My Projects | No | No | Yes | Yes |
| Trending Management | No | No | No | Yes |
| Project Approval | No | No | No | Yes |
| Account Settings | No | Yes | Yes | Yes |
