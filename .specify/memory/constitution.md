# Project Constitution

## Core Principles

### I. Monorepo-First Architecture

The entire system MUST live in a single production-grade monorepo managed with **Turborepo** for
task orchestration and **pnpm workspaces** for dependency management. Every app and package MUST be
independently buildable and testable in isolation.

- Turborepo MUST be configured with incremental builds and remote caching.
- pnpm workspaces MUST be the sole package manager; npm and yarn are prohibited.
- All apps reside under `/apps/`; all shared libraries under `/packages/`.
- A package MUST NOT import from an app (no reverse dependencies).
- Repository structure MUST match:

```
/apps
  /<web-app-name>        # Each deployable web application
  /<mobile-app-name>     # Each React Native mobile application

/packages
  /ui-components         # Shared React web UI library (Storybook, WCAG)
  /mobile-ui             # Shared React Native UI components
  /shared-types          # DTOs, enums, Zod schemas, API contracts
  /api-client            # Typed HTTP client consumed by all apps
  /config                # Shared env constants, design token primitives, tsconfig
  /eslint-config         # Shared ESLint ruleset
```

**Rationale**: A single monorepo eliminates version drift between packages, enforces shared
contracts, and enables atomic cross-package refactors.

### II. Clean Architecture & SOLID

Both the backend and all frontend/mobile applications MUST follow **Clean Architecture** and
**SOLID** principles. The dependency rule is absolute: domain layer → application layer →
infrastructure layer. The direction is never reversed.

**Dependency Rule (non-negotiable)**:

- Domain/business logic MUST NOT import from infrastructure (databases, HTTP, external services).
- Application layer orchestrates domain objects; it MUST NOT contain infrastructure concerns.
- Infrastructure adapts external systems to application interfaces; it MUST NOT contain business
  rules.

**Backend (NestJS) behavioral contracts**:

- Business logic MUST reside in service classes; controllers MUST only parse requests and
  delegate — no conditionals, no domain rules in controllers.
- Services MUST be stateless; no request-scoped mutable state in services.
- Cross-cutting concerns MUST use Guards (auth/RBAC), Interceptors (logging, transformation),
  Pipes (validation), and Exception Filters (error normalisation).
- Custom decorators MUST be used for repetitive metadata concerns (roles, current user, etc.).
- Module boundaries MUST map to business domains (domain-driven modules).

**Frontend (React) behavioral contracts**:

- No business rules MUST be hardcoded in UI components.
- Business logic MUST NOT leak into JSX render functions — extract to hooks or services.
- Data fetching MUST go through the api-client package; raw fetch/axios in app code is prohibited.

**Mobile (React Native) behavioral contracts**:

- Business logic MUST NOT leak into screen components or navigation configuration.
- Platform-specific code MUST be isolated in `.ios.ts` / `.android.ts` suffixed files or a
  `platform/` directory within the feature folder.
- Data fetching MUST go through the api-client package; raw fetch/axios in app code is prohibited.

**SOLID rules (all platforms)**:

- **Single Responsibility**: each module, service, and component MUST have one reason to change.
- **Open/Closed**: extend behaviour via composition and injection, not modification of existing
  code.
- **Liskov Substitution**: subtypes MUST honour the contracts of their parent types.
- **Interface Segregation**: inject narrow interfaces, not fat abstract classes.
- **Dependency Inversion**: depend on abstractions (interfaces, DI tokens); never on concrete
  implementations directly.

**Rationale**: Clean boundaries make the system testable in isolation, maintainable as the team
scales, and refactorable without full rewrites. SOLID principles prevent the accumulation of
complexity that makes systems brittle.

### III. Modular Architecture

Every application (backend, frontend, mobile) MUST be structured as a collection of self-contained,
domain-driven modules. A module encapsulates all code for a single business domain and exposes a
defined public API. Cross-module imports MUST only use the module's public API (barrel export);
reaching into a module's internal files is prohibited.

**This principle is the canonical authority for all folder structure decisions. When in doubt about
where a file belongs, consult Principle III.**

**Backend (NestJS) module rules**:

- Each NestJS module MUST map to exactly one business domain (e.g., `users`, `orders`, `payments`).
- A module directory MUST contain: `<domain>.module.ts`, `<domain>.controller.ts`,
  `<domain>.service.ts`, DTOs (`dto/`), entities (`entities/`), and an `index.ts` barrel export.
- Inter-module communication MUST use injected services or events; direct database access across
  module boundaries is prohibited.
- Shared infrastructure modules (database, auth, logging) MUST live under a `common/`
  directory and be imported explicitly — no global implicit providers.
- Module structure MUST match:

```
src/modules/<domain>/
├── <domain>.module.ts
├── <domain>.controller.ts
├── <domain>.service.ts
├── dto/
├── entities/
├── guards/           # (optional) domain-specific guards
├── interceptors/     # (optional) domain-specific interceptors
├── __tests__/
└── index.ts          # public API barrel export
```

**Frontend (React) module rules**:

- Each feature module MUST be a self-contained directory under `src/features/<feature>/`.
- A feature module MUST contain: `ui/` (pages and components), `state/` (Zustand stores),
  `services/` (API calls via api-client), `hooks/`, and an `index.ts` barrel export.
- Cross-feature imports MUST go through the barrel export; importing from another feature's internal
  `ui/` or `state/` directory is prohibited.
- Shared UI primitives (buttons, inputs, layout) MUST live in `/packages/ui-components`, not inside
  a feature module.
- Feature module structure MUST match:

```
src/features/<feature>/
├── ui/
│   ├── pages/        # route-level page components
│   └── components/   # feature-scoped components
├── state/            # Zustand stores
├── services/         # API calls via api-client
├── hooks/            # feature-scoped custom hooks
├── domain/           # models, types, validation
├── __tests__/
└── index.ts          # public API barrel export
```

**Mobile (React Native) module rules**:

- Each feature module MUST be a self-contained directory under `src/features/<feature>/`.
- A feature module MUST contain: `ui/` (screens and components), `state/` (stores), `services/`
  (API calls via api-client), `hooks/`, `navigation/` (feature-scoped navigators), and an `index.ts`
  barrel export.
- Navigation configuration (route definitions, navigator setup) MUST live in the feature's
  `navigation/` subdirectory, not in screen components.
- Platform-specific files (`.ios.ts`, `.android.ts`) MUST NOT be re-exported from barrel exports
  (`index.ts`). Platform file resolution is handled by the React Native bundler at build time. The
  barrel export MUST reference only the base file name (e.g., `export { CameraButton } from
  './CameraButton'` — the bundler resolves `.ios.ts` or `.android.ts` automatically).
- Cross-feature imports MUST go through the barrel export; importing from another feature's
  internals is prohibited.
- Shared mobile UI primitives MUST live in `/packages/mobile-ui`, not inside a feature module.
- Feature module structure MUST match:

```
src/features/<feature>/
├── ui/
│   ├── screens/      # screen-level components
│   └── components/   # feature-scoped components
├── state/            # Zustand stores
├── services/         # API calls via api-client
├── hooks/            # feature-scoped custom hooks
├── navigation/       # feature-scoped navigators and routes
├── domain/           # models, types, validation
├── platform/         # (optional) iOS/Android specific code
├── __tests__/
└── index.ts          # public API barrel export
```

**Cross-platform module rules**:

- Module names MUST be consistent across backend, frontend, and mobile when they represent the same
  business domain (e.g., `users/` in all three).
- New modules MUST be registered in the application's root configuration (NestJS app module, React
  router, React Native navigator) before use.
- A module MUST NOT exceed a reasonable size; when a module grows unwieldy, it SHOULD be
  evaluated for decomposition into sub-modules.
- Circular dependencies between modules are prohibited; if detected, extract shared logic into a
  new module or shared package.

**Rationale**: Modular architecture enforces separation of concerns at the feature level, enables
parallel team development without merge conflicts, makes features independently testable and
removable, and provides a consistent mental model across all platforms.

### IV. Strict Type Safety (NON-NEGOTIABLE)

TypeScript strict mode MUST be enabled in every package and app. Zero `any` is the enforced
standard.

- `tsconfig.json` with `"strict": true`, `"noImplicitAny": true`, `"strictNullChecks": true` MUST be
  shared from `/packages/config`.
- `/packages/shared-types` is the **single source of truth** for all DTOs, enums, API response
  shapes, and Zod validation schemas.
- No type assertion (`as SomeType`) is permitted without an inline justification comment reviewed in
  PR.
- Barrel exports (`index.ts`) MUST be used at package boundaries.
- CI MUST run `tsc --noEmit` on every package; type errors block merge.

**Rationale**: Shared types eliminate contract drift between frontend and backend. Strict mode
catches entire classes of runtime bugs at compile time.

### V. Security by Design

Security controls MUST be applied at the framework layer, not ad-hoc per endpoint.

- Authentication: JWT (access + refresh token pattern) enforced via a global Auth Guard; all routes
  are protected by default; public routes MUST be explicitly marked.
- Authorisation: Role-Based Access Control (RBAC) via NestJS role guards using roles defined in
  `/packages/shared-types`; role checks MUST never live in service logic.
- Rate limiting MUST be applied globally with configurable per-route overrides.
- CORS MUST be explicitly configured; wildcard origins (`*`) are prohibited in staging and
  production.
- `helmet` middleware MUST be mounted globally on the NestJS app.
- All incoming data MUST be validated via `class-validator` DTOs before reaching service logic; raw
  request bodies MUST never be passed to services.
- Secrets MUST be injected via environment variables; no secrets in source code or committed config
  files.
- Mobile apps MUST store sensitive tokens using **react-native-keychain**; plain AsyncStorage is
  prohibited for secrets.

**Rationale**: Security controls embedded at the framework level are harder to accidentally bypass
than per-endpoint implementations.

### VI. Testing Discipline

All packages and apps MUST include automated tests. The test suite is a first-class deliverable, not
an afterthought.

- Unit tests (Jest) are REQUIRED for all service classes, utility functions, and stores.
- Integration tests are REQUIRED for all API endpoints and cross-module interactions.
- **Minimum 80% code coverage is enforced in CI** — PRs that drop coverage below this threshold MUST
  NOT be merged.
- Contract tests MUST validate `/packages/shared-types` schemas against real API responses.
- Frontend component tests MUST use React Testing Library.
- Mobile component tests MUST use React Native Testing Library.
- Mobile apps MUST include **Maestro** E2E tests for critical user flows.
- Tests MUST be co-located with source files or in a sibling `__tests__/` directory.

**Rationale**: High coverage with meaningful integration tests provides a regression safety net
sufficient for production systems.

### VII. Independent Deployability

Every application MUST be deployable independently without touching other apps.

- Each web/backend app MUST have its own multi-stage `Dockerfile` producing a production-optimised
  image.
- Mobile apps MUST have independent build pipelines using **EAS Build** for iOS and Android.
- The backend MUST be stateless; session state MUST NOT be stored in memory.
- All configuration MUST be injected via environment variables (Twelve-Factor App compliance).
- Health check (`/health`), readiness (`/ready`), and liveness (`/live`) endpoints MUST be present
  on the backend.
- CI/CD MUST use GitHub Actions with branch environments: `dev`, `staging`, `production`.
- Pipeline stages MUST run in order: lint → type-check → test (with coverage) → build → Docker image
  → push to registry (backend/web); lint → type-check → test → build → submit to stores (mobile).
- Frontend apps MUST use lazy loading and code splitting; no unneeded bundle weight.

**Rationale**: Independent deployability allows teams to ship, roll back, and scale individual apps
without coordinating a full-platform release.

### VIII. Observability-First

Every service MUST emit actionable signals for debugging, alerting, and capacity planning from day
one.

- Structured JSON logging MUST be implemented via **Pino** (via `nestjs-pino`) on the backend;
  plain `console.log` is prohibited in production code.
- All log entries MUST include: timestamp, level, service name, correlation/request ID, and relevant
  context (user ID, resource ID).
- Error monitoring MUST use **Sentry** (SDK integrated, DSN via env var).
- A Prometheus-compatible `/metrics` endpoint MUST be exposed by the backend.
- Frontend and mobile apps MUST integrate Sentry for uncaught exception capture.
- Mobile apps MUST report crash analytics and ANR/hang diagnostics via Sentry.
- Every route-level page component MUST be wrapped in an Error Boundary.
- Error boundaries MUST report to Sentry and render a user-friendly fallback.
- Mobile: screen crashes MUST NOT crash the entire app.

**Rationale**: Observability is not optional in production systems; undiagnosed failures have
operational and business consequences.

### IX. Shared-Before-Custom

Code that can be shared MUST be shared. Duplication of logic across app boundaries is prohibited.

- Web UI components used by more than one app MUST live in `/packages/ui-components`, be documented
  in Storybook, meet WCAG accessibility standards, and contain zero business logic.
- Mobile UI components shared across mobile apps MUST live in `/packages/mobile-ui` and contain zero
  business logic.
- All API request/response types MUST be defined in `/packages/shared-types` before being used in
  any app.
- All apps (web and mobile) MUST consume the backend exclusively through `/packages/api-client`
  (typed HTTP client with centralised token management and error handling); direct `fetch` or raw
  Axios usage in app code is prohibited.
- `/packages/ui-components` and `/packages/mobile-ui` MUST NOT import from any app or from
  `api-client`.

**Rationale**: Centralised contracts and UI eliminate the divergence that plagues multi-app
platforms and reduce the surface area for bugs at integration points.

### X. Design Token Management

All visual styling properties MUST be defined as variables (design tokens) in a centralised theme
layer. Hardcoding raw colour values, font families, font sizes, spacing scales, border radii,
shadows, or breakpoints directly in component code is prohibited.

**Token source of truth**:

- All design token primitives (colour scales, type scales, spacing scales) MUST be defined in
  `/packages/config/src/theme/`.
- `/packages/ui-components` MUST re-export the Tailwind theme configuration, consuming token values
  from `/packages/config`.
- `/packages/mobile-ui` MUST re-export the `theme.ts` object, consuming token values from
  `/packages/config`.
- Web apps MUST consume the Tailwind configuration from `/packages/ui-components`.
- Mobile apps MUST consume the theme object from `/packages/mobile-ui`.

**Web frontend apps (React + Tailwind CSS)**:

- All colours MUST be defined as CSS custom properties (e.g., `--color-primary`) or Tailwind
  `theme.extend` tokens; raw hex/rgb/hsl values in component files are prohibited.
- All font families and font sizes MUST be defined as Tailwind theme tokens or CSS custom
  properties; inline `font-family` or arbitrary `text-[16px]` values are prohibited unless mapped to
  a named token.
- Spacing, border-radius, and shadow values MUST reference the shared scale; magic numbers are
  prohibited.
- Theme tokens MUST support light/dark mode variants where applicable via CSS custom properties or
  Tailwind's `dark:` variant bound to token values.

**Mobile apps (React Native)**:

- Direct use of hardcoded colour strings (e.g., `color: '#FF0000'`) in StyleSheet definitions or
  inline styles is prohibited; all values MUST reference the theme object.
- Theme tokens MUST support light/dark mode variants via React Native's `Appearance` API or a
  theme provider.

**Enforcement**:

- ESLint rules (e.g., `no-restricted-syntax` or a custom plugin) MUST flag raw colour/font
  literals in TSX/JSX files.
- PR reviews MUST reject hardcoded visual values that bypass the token system.

**Rationale**: Centralised design tokens enable consistent theming across all apps, simplify brand
updates and dark-mode support, and prevent visual drift between independently developed
applications.

## Naming Conventions

All naming conventions below are non-negotiable and enforced by ESLint rules and code review.
When a rule conflicts with a third-party library's API (e.g., database column naming required by
an ORM), the convention applies to the project's own code; the exception MUST be documented inline.

### File Naming

- **TypeScript source files**: `kebab-case.ts` — e.g., `user-profile.service.ts`,
  `create-order.dto.ts`, `auth.module.ts`.
- **React component files** (web and mobile): `PascalCase.tsx` — e.g., `UserCard.tsx`,
  `OrderSummaryPanel.tsx`. The file name MUST match the exported component name.
- **Test files**: co-located, named `<source-file>.spec.ts` — e.g., `user.service.spec.ts`,
  `UserCard.spec.tsx`.
- **Index barrel files**: always `index.ts` — never `index.tsx` unless the barrel itself exports
  JSX (which it MUST NOT; barrels export named re-exports only).
- **Configuration files**: `kebab-case.config.ts` — e.g., `jest.config.ts`, `tailwind.config.ts`.
- **Migration files**: Prisma-generated; do not rename. Format is
  `<timestamp>_<descriptive_name>` (Prisma default).

### Component and Class Naming

- React components (web and mobile): **PascalCase** — e.g., `UserAvatar`, `CheckoutForm`.
- NestJS modules, controllers, services, guards, interceptors, pipes, filters: **PascalCase**
  with suffix — `UsersModule`, `UsersController`, `UsersService`, `RolesGuard`,
  `LoggingInterceptor`, `ParseIntPipe`, `HttpExceptionFilter`.
- Zod schemas: **PascalCase** with `Schema` suffix — e.g., `CreateUserSchema`,
  `OrderResponseSchema`.
- TypeScript interfaces: **PascalCase**, no `I` prefix — e.g., `UserProfile`, `ApiResponse`.
- TypeScript type aliases: **PascalCase** — e.g., `UserId`, `PaginationMeta`.
- TypeScript enums: **PascalCase** enum name, **SCREAMING_SNAKE_CASE** members — e.g.,
  `enum UserRole { ADMIN = 'ADMIN', MEMBER = 'MEMBER' }`.

### Variable and Function Naming

- Variables and function names: **camelCase** — e.g., `userId`, `fetchUserById`,
  `isAuthenticated`.
- Boolean variables and props: MUST use an `is`, `has`, `can`, or `should` prefix — e.g.,
  `isLoading`, `hasPermission`, `canEdit`, `shouldRetry`.
- Constants (module-level, immutable): **SCREAMING_SNAKE_CASE** — e.g., `MAX_RETRY_COUNT`,
  `DEFAULT_PAGE_SIZE`.
- Private class members: no underscore prefix; use TypeScript `private` keyword.
- Event handler props on React components: `on` prefix — e.g., `onSubmit`, `onChange`,
  `onUserSelect`.
- Custom hooks: MUST begin with `use` — e.g., `useCurrentUser`, `useOrderList`.
- Zustand store hooks: `use<Domain>Store` — e.g., `useAuthStore`, `useCartStore`.

### Import Ordering

ESLint (`eslint-plugin-simple-import-sort`) MUST enforce this order, with a blank line between
each group:

1. Node built-in modules (`fs`, `path`, `crypto`)
2. External packages (`react`, `@nestjs/common`, `zod`)
3. Internal monorepo packages (`@<project>/shared-types`, `@<project>/api-client`)
4. Local absolute imports within the same app (`@/features/auth`, `@/common/guards`)
5. Local relative imports (`./user.service`, `../dto/create-user.dto`)

Type-only imports (`import type { ... }`) MUST use `import type` syntax and be grouped at the end
of the relevant group. Alphabetical order within each group is REQUIRED. No mixing of groups.

### API Endpoint Naming (REST)

- All routes MUST be prefixed with `/v1` (enforced by global prefix in NestJS bootstrap).
- Resource names MUST be **plural nouns in kebab-case** — e.g., `/v1/users`, `/v1/order-items`,
  `/v1/payment-methods`.
- Nested resources use path nesting: `/v1/users/:userId/addresses`.
- Actions that do not map to CRUD MUST use a verb suffix on the resource: `/v1/auth/login`,
  `/v1/auth/logout`, `/v1/users/:userId/deactivate`.
- Query parameters MUST be **camelCase** — e.g., `?pageSize=20&sortBy=createdAt`.
- Response envelope MUST follow: `{ success: boolean, data: T | null, error?: { code: string, message: string, details?: Record<string, string[]> },
  meta?: PaginationMeta }`.
- HTTP method semantics MUST be respected: `GET` for reads, `POST` for create, `PATCH` for partial
  update, `PUT` for full replace, `DELETE` for deletion. Using `POST` as a catch-all is prohibited.

### Database Naming (PostgreSQL / Prisma)

- Table names: **snake_case, plural** — configured via `@@map("table_name")` in Prisma schema.
  E.g., `users`, `order_items`, `payment_methods`.
- Column names: **snake_case** — configured via `@map("column_name")` in Prisma schema. E.g.,
  `created_at`, `user_id`, `is_active`.
- Prisma model names (TypeScript side): **PascalCase singular** — e.g., `User`, `OrderItem`.
  The `@@map` directive maps them to snake_case table names.
- Primary keys: always named `id`, type `String` with `@default(cuid())` unless a specific domain
  reason requires otherwise (must be justified in PR).
- Foreign key columns: `<relation_name>_id` in snake_case — e.g., `user_id`, `order_id`.
- Boolean columns: prefixed with `is_` or `has_` — e.g., `is_active`, `has_verified_email`.
- Timestamp columns: `created_at`, `updated_at`, `deleted_at` (for soft delete).
- Indexes MUST be named explicitly: `@@index([userId], name: "idx_orders_user_id")`.
- Enums in Prisma: **PascalCase** name, **SCREAMING_SNAKE_CASE** values, must mirror the
  TypeScript enum in `/packages/shared-types`.

### Database Migration Safety

- Migrations MUST be backward-compatible in production.
- Destructive migrations (drop table/column, rename) MUST be in separate PRs with a deprecation
  period.
- Seeds MUST be idempotent.
- Migration files MUST NOT be edited after merge to main.

### Environment Variables

- Environment variable names MUST use `SCREAMING_SNAKE_CASE` with an app prefix — e.g.,
  `APP_DATABASE_URL`, `APP_JWT_SECRET`.
- `.env.example` MUST be committed with placeholder values; `.env` MUST be gitignored.
- No secrets in `.env.example`.
- Configuration validation: `@nestjs/config` + Zod schema at startup; the app MUST crash on
  invalid config.

### Branch Naming

- Feature branches: `<issue-number>-<short-description-in-kebab-case>` — e.g.,
  `042-user-authentication`, `108-order-export-csv`.
- Bug fix branches: `fix/<issue-number>-<short-description>` — e.g.,
  `fix/097-login-redirect-loop`.
- Chore/maintenance branches: `chore/<short-description>` — e.g., `chore/upgrade-nestjs-10`.
- Branch names MUST be lowercase kebab-case. No spaces, no uppercase, no underscores.

### Commit Message Conventions

Conventional Commits format: `<type>(<scope>): <imperative description>`

**Types**: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `ci`

**Scope values** (pick the most specific):

| Scope | Applies to |
|-------|------------|
| `backend` | NestJS app code |
| `web` | Web frontend app(s) |
| `mobile` | React Native app(s) |
| `shared-types` | `/packages/shared-types` |
| `api-client` | `/packages/api-client` |
| `ui-components` | `/packages/ui-components` |
| `mobile-ui` | `/packages/mobile-ui` |
| `config` | `/packages/config`, `/packages/eslint-config` |
| `infra` | Docker, CI/CD, GitHub Actions, EAS config |
| `db` | Prisma schema, migrations, seeds |
| `constitution` | `.specify/memory/constitution.md` amendments |

**Description rules**: imperative mood, lowercase first letter, no period at end.

Good: `feat(backend): add rate limiting to auth endpoints`
Bad: `Fixed the login bug`, `Added feature`, `WIP`

### Documentation

- Each `/packages/` package MUST have a `README.md` covering: purpose, install instructions, usage
  examples, and API reference.
- Public functions in shared packages SHOULD have TSDoc comments.
- No inline docs for self-explanatory code.

## Error Handling Conventions

All error handling MUST follow a consistent, typed pattern across the entire stack.

- Custom exception hierarchy on the backend: `BusinessException`, `ValidationException`,
  `NotFoundException` — all extending a base `AppException`.
- A Global Exception Filter MUST catch all exceptions and map them to the standard response
  envelope.
- HTTP status code mapping MUST be consistent:
  - `400` — validation errors
  - `401` — authentication failures
  - `403` — authorisation failures
  - `404` — resource not found
  - `409` — conflict (duplicate resource, optimistic lock)
  - `422` — business rule violations
- Frontend and mobile: the `api-client` package MUST normalise all API errors into a typed
  `ApiError` class with `code`, `message`, and optional `details`.

## State Management Conventions

- One store per domain/feature, named `use<Domain>Store` (see Variable and Function Naming).
- **TanStack Query** for all server state (API data fetching and caching); **Zustand** for
  client-only state (UI state, form drafts, local preferences) — never mix the two.
- Store actions MUST be defined inside the store; external mutation of store state is prohibited.
- No derived state in stores — use selectors or computed values outside the store.
- Persist middleware (e.g., `zustand/middleware`) MUST only be used for explicit offline or cache
  use cases; never persist ephemeral UI state.

## Technology Stack

### Backend

| Concern           | Choice                                    |
| ----------------- | ----------------------------------------- |
| Framework         | NestJS (latest LTS)                       |
| Language          | TypeScript (strict mode)                  |
| Database          | PostgreSQL                                |
| ORM               | Prisma                                    |
| API documentation | Swagger (OpenAPI 3.0) via `@nestjs/swagger` |
| Logging           | Pino (via `nestjs-pino`)                  |
| Testing           | Jest (unit + integration)                 |
| Containerisation  | Docker (multi-stage)                      |
| API versioning    | `/v1` prefix on all routes                |
| Response format   | `{ success, data, error?, meta? }`        |

### Web Frontend Apps

| Concern          | Choice                                 |
| ---------------- | -------------------------------------- |
| Framework        | React.js (latest stable)               |
| Language         | TypeScript (strict mode)               |
| Styling          | Tailwind CSS                           |
| Design tokens    | CSS custom properties + Tailwind theme |
| State management | Zustand                                |
| Data fetching    | TanStack Query (React Query)           |
| Testing          | Jest + React Testing Library           |

### Mobile Apps

| Concern          | Choice                              |
| ---------------- | ----------------------------------- |
| Framework        | React Native (latest stable)        |
| Language         | TypeScript (strict mode)            |
| State management | Zustand                             |
| Design tokens    | Shared theme object (`theme.ts`)    |
| Navigation       | React Navigation                    |
| Testing          | Jest + React Native Testing Library |
| E2E testing      | Maestro                             |
| Secure storage   | react-native-keychain               |
| CI builds        | EAS Build                           |

### Shared Packages

| Package                   | Responsibility                                              |
| ------------------------- | ----------------------------------------------------------- |
| `/packages/ui-components` | Reusable React web components; Storybook; WCAG              |
| `/packages/mobile-ui`     | Reusable React Native components; shared theme              |
| `/packages/shared-types`  | DTOs, enums, Zod schemas, API contracts                     |
| `/packages/api-client`    | Typed HTTP client; token management; error handling         |
| `/packages/config`        | Shared env constants; design token primitives; tsconfig     |
| `/packages/eslint-config` | Shared ESLint ruleset (strict)                              |

## Code Quality Standards

The following standards are non-negotiable and enforced by tooling:

**Tooling (enforced pre-commit via Husky)**:

- ESLint with strict rules from `/packages/eslint-config` — zero warnings policy.
- Prettier for formatting — no style deviations permitted.
- Conventional Commits with mandatory scope (see Naming Conventions).
- TypeScript compilation check (`tsc --noEmit`) runs before commit on changed packages.
- Import ordering enforced by `eslint-plugin-simple-import-sort`.

**Design rules**:

- DRY: identical logic MUST NOT exist in more than one package.
- Dependency Injection MUST be used on the backend (NestJS IoC container).
- No business rules in UI layers (see Principle II).
- No hardcoded visual values (colours, fonts, spacing) — all MUST reference design tokens
  (see Principle X).
- All code MUST follow the Modular Architecture folder structure (see Principle III).
- Barrel exports (`index.ts`) at every public package/module boundary.
- Cross-module imports MUST use barrel exports only; internal file imports are prohibited.

### Performance Standards

- Frontend apps MUST use React lazy loading and route-level code splitting.
- Mobile apps MUST use lazy screens; unnecessary re-renders MUST be identified via React DevTools
  and eliminated (`React.memo`, `useMemo`) before merge.
- Backend MUST paginate all list endpoints; unbounded queries are prohibited.
- Database indexes MUST be defined alongside their corresponding schema migrations.
- A Redis caching layer MUST be designed for (connection configured) even if not initially
  populated; no in-process caching that breaks horizontal scale.
- The backend MUST be stateless and horizontally scalable by design.

### Accessibility Standards

- All interactive elements MUST have accessible labels (`aria-label`, `accessibilityLabel`).
- Colour contrast MUST meet WCAG AA (4.5:1 for normal text, 3:1 for large text).
- All forms MUST support keyboard navigation (web) and screen reader announcements (mobile).
- Lighthouse accessibility score MUST be >= 90 in CI (web apps).

### Feature Flags

- Feature flags MUST use environment variables or a dedicated feature flag service.
- Flag names MUST follow `SCREAMING_SNAKE_CASE` with `FF_` prefix — e.g., `FF_NEW_CHECKOUT_FLOW`.
- Dead flags (fully rolled out) MUST be removed within one sprint of full rollout.

## Governance

This constitution supersedes all other development practices, ADRs, or verbal agreements. When a
conflict exists between this document and any other guideline, this constitution takes precedence.

**Amendment procedure**:

1. Open a PR with changes to `.specify/memory/constitution.md`.
2. Bump `CONSTITUTION_VERSION` following semantic versioning:
   - MAJOR — removal or backward-incompatible redefinition of a principle.
   - MINOR — new principle or section added, or materially expanded guidance.
   - PATCH — clarification, wording fix, or non-semantic refinement.
3. Update `LAST_AMENDED_DATE` to the date of merge.
4. Propagate changes to affected `.specify/templates/` files in the same PR.
5. At least one other team member MUST review constitutional changes.

**Compliance**:

- All PRs MUST pass the CI pipeline (lint, type-check, test, coverage >= 80%, build).
- Before implementation begins on any feature, the assigned developer MUST complete the
  Constitution Check gate in the feature's `plan.md`. The gate is not advisory — it blocks
  implementation.
- The `speckit.analyze` command MUST be run before a PR is opened; it performs automated
  cross-artifact consistency checking against this constitution.
- Violations of hard constraints (e.g., `any` usage, business logic in controllers, skipped tests,
  hardcoded colours, cross-module internal imports) MUST be flagged in code review and corrected
  before merge.

### PR & Code Review Workflow

- Squash merge to main for linear history.
- PR title MUST follow Conventional Commits format (see Commit Message Conventions).
- Minimum 1 approval required before merge.
- Draft PRs for work-in-progress; mark ready-for-review only when CI passes.
- PR description MUST reference the issue number and include a test plan.

### Monorepo Versioning

- All packages in the monorepo MUST use **fixed versioning** — the same version number, bumped
  together on release.
- Changesets or a Turborepo release pipeline MUST coordinate version bumps.

**Version**: 2.1.0 | **Ratified**: 2026-03-04 | **Last Amended**: 2026-03-04
