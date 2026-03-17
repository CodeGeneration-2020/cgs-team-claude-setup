# cgs-team-claude-setup Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-03-17

## Active Technologies
- PostgreSQL (users, roles, password reset tokens, audit logs) (002-registration-roles)
- PostgreSQL (projects, taxonomy, audit logs) (003-project-management)
- PostgreSQL (bookmarks table, bookmark_updates derived from project_audit_log) (004-bookmarks)
- PostgreSQL (user_preferences table) (005-onboarding)
- PostgreSQL (notifications table, notification_preferences on user) (006-notifications)
- TypeScript (strict mode) + NestJS (backend), React.js + Tailwind CSS (frontend), Zustand + TanStack Query (state/data), Prisma ORM, LLM API (for contextual responses and answer-to-query matching) (007-ai-assistant)
- PostgreSQL (assistant_questions, assistant_sessions for authenticated users) (007-ai-assistant)
- TypeScript (strict mode) + NestJS (backend — minimal changes), React.js + Tailwind CSS (frontend), map library (Leaflet or Mapbox GL JS), geocoding service (008-project-map)
- PostgreSQL — optional latitude/longitude cache columns on projec (008-project-map)
- TypeScript (strict mode) + NestJS (backend), React.js + Tailwind CSS (frontend), Prisma ORM, vector store (pgvector from 001-homepage) (009-matchmaking)
- PostgreSQL (match_scores cache table optional, preference embeddings) (009-matchmaking)

- TypeScript (strict mode) + NestJS (backend), React.js + Tailwind CSS (frontend), Zustand + TanStack Query (state/data), Prisma ORM (001-homepage)

## Project Structure

```text
src/
tests/
```

## Commands

npm test && npm run lint

## Code Style

TypeScript (strict mode): Follow standard conventions

## Recent Changes
- 009-matchmaking: Added TypeScript (strict mode) + NestJS (backend), React.js + Tailwind CSS (frontend), Prisma ORM, vector store (pgvector from 001-homepage)
- 008-project-map: Added TypeScript (strict mode) + NestJS (backend — minimal changes), React.js + Tailwind CSS (frontend), map library (Leaflet or Mapbox GL JS), geocoding service
- 007-ai-assistant: Added TypeScript (strict mode) + NestJS (backend), React.js + Tailwind CSS (frontend), Zustand + TanStack Query (state/data), Prisma ORM, LLM API (for contextual responses and answer-to-query matching)


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
