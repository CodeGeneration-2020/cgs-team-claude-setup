# Quickstart: Project Management

**Feature**: 003-project-management | **Date**: 2026-03-17

## Prerequisites

- Node.js (LTS), pnpm
- PostgreSQL instance
- Features 001-homepage and 002-registration-roles implemented (search, auth, RBAC)
- Client-provided taxonomy seed data file

## Setup

```bash
pnpm install
pnpm --filter <backend-app> prisma generate
pnpm --filter <backend-app> prisma migrate dev    # adds project fields, taxonomy_values, project_audit_log tables
pnpm --filter <backend-app> prisma db seed         # seeds taxonomy values + Super Admin
```

## Development

```bash
pnpm dev
```

## Testing

```bash
# Backend project module
pnpm --filter <backend-app> test -- --testPathPattern=projects

# Backend taxonomy module
pnpm --filter <backend-app> test -- --testPathPattern=taxonomy

# Backend admin review module
pnpm --filter <backend-app> test -- --testPathPattern=admin-review

# Frontend project management
pnpm --filter <web-app> test -- --testPathPattern=project-management

# Frontend admin review
pnpm --filter <web-app> test -- --testPathPattern=admin-review

# E2E
pnpm --filter <web-app> test:e2e -- --testPathPattern=project-management
```

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /v1/projects | ORG_ADMIN | Create project |
| GET | /v1/projects/my | ORG_ADMIN | List my projects |
| GET | /v1/projects/:id | Public/Owner | Get project detail |
| PATCH | /v1/projects/:id | ORG_ADMIN (owner) | Update project |
| POST | /v1/projects/:id/publish | ORG_ADMIN (owner) | Publish for review |
| DELETE | /v1/projects/:id | ORG_ADMIN (owner) | Delete project |
| GET | /v1/taxonomy | Public | Get taxonomy values |
| GET | /v1/admin/projects/pending | SUPER_ADMIN | Pending review queue |
| POST | /v1/admin/projects/:id/approve | SUPER_ADMIN | Approve project |
| POST | /v1/admin/projects/:id/reject | SUPER_ADMIN | Reject with reason |

## Key Modules

**Backend:** `projects` (CRUD), `taxonomy` (predefined values), `admin-review` (approval queue)
**Frontend:** `project-management` (Org Admin pages), `admin-review` (Super Admin queue)
**Shared:** project types, taxonomy types, admin types in `packages/shared-types`
