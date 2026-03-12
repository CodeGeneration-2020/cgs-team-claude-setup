# CGS Team Claude Setup

A complete AI-assisted development workflow powered by [Claude Code](https://claude.com/claude-code), [Spec Kit](https://github.com/lsendel/spec-kit-mcp), and the Model Context Protocol (MCP).

This repo provides the full infrastructure for spec-driven development: from writing feature specs, through planning and implementation, to visual validation against Figma designs.

---

## Table of Contents

- [Quick Start](#quick-start)
- [MCP Servers](#mcp-servers)
- [Agents](#agents)
- [Skills / Commands](#skills--commands)
  - [Specification Workflow](#specification-workflow)
  - [Figma Workflow](#figma-workflow)
- [Workflow Overview](#workflow-overview)
- [Real-World Examples](#real-world-examples)
  - [Example 1: Setting Up Spec Kit on an Existing Project](#example-1-setting-up-spec-kit-on-an-existing-project)
  - [Example 2: Converting Existing Docs and Code into Specs](#example-2-converting-existing-docs-and-code-into-specs)
  - [Example 3: Capturing App Screens into Figma](#example-3-capturing-app-screens-into-figma)
- [Project Structure](#project-structure)

---

## Quick Start

1. **Install Claude Code** — follow the [official guide](https://docs.anthropic.com/en/docs/claude-code)
2. **Clone this repo** and open it in your terminal
3. **Start Claude Code** — MCP servers defined in `.mcp.json` will be loaded automatically

```bash
git clone <repo-url>
cd cgs-team-claude-setup
claude
```

All skills, agents, and MCP servers are configured at the project level and available immediately.

---

## MCP Servers

Configured in `.mcp.json`. These extend Claude Code with external tool capabilities.

| Server | Package | Purpose |
|--------|---------|---------|
| **spec-kit** | `@lsendel/spec-kit-mcp` | Spec Kit workflow — templates, scripts, and feature scaffolding |
| **playwright** | `@anthropic-ai/mcp-server-playwright` | Browser automation — navigate pages, fill forms, take screenshots, run accessibility checks |
| **figma** | `@anthropic-ai/mcp-server-figma` | Figma integration — read designs, extract metadata, manage Code Connect mappings |
| **chrome-devtools** | `@anthropic-ai/mcp-server-chrome-devtools` | Chrome DevTools — inspect network requests, console errors, evaluate JS, performance tracing |

### Usage

MCP tools are available to Claude Code and agents automatically. You don't invoke them directly — they're used behind the scenes when agents need to interact with browsers, Figma, or the spec kit system.

To verify servers are loaded, check the status bar in Claude Code or run `/mcp` in the CLI.

---

## Agents

Located in `.claude/agents/`. These are specialized AI personas that Claude Code can delegate work to.

### Backend Developer

| | |
|---|---|
| **File** | `.claude/agents/backend-developer.md` |
| **Focus** | NestJS API development — controllers, services, Prisma schema/migrations, DTOs, guards, interceptors |
| **Tools** | Read, Write, Edit, Bash, Grep, Glob, Task + Chrome DevTools MCP |
| **Modes** | **tasks.md-driven** (processes backend tasks in order) or **ad-hoc** (direct instructions) |

Invoke directly:

```
@backend-developer add the projects CRUD endpoints
```

Or let `/speckit.implement` delegate backend tasks to it automatically.

### Web Developer

| | |
|---|---|
| **File** | `.claude/agents/web-developer.md` |
| **Focus** | Frontend UI — implement pages and components from Figma designs, write unit tests, validate visually |
| **Tools** | Read, Write, Edit, Bash, Grep, Glob, Task + Figma MCP + Playwright MCP |
| **Modes** | **tasks.md-driven** (processes frontend tasks in order) or **ad-hoc** (direct instructions) |

Invoke directly:

```
@web-developer implement the users page from this Figma link: https://figma.com/design/...
```

---

## Skills / Commands

Located in `.claude/commands/`. Invoke them by typing the command name in Claude Code (e.g., `/speckit.specify`).

### Specification Workflow

These skills follow a sequential pipeline. Each builds on the output of the previous step.

#### `/speckit.specify` — Create Feature Spec

Creates a new feature branch and generates `spec.md` from a natural language description.

```
/speckit.specify User authentication with email/password and OAuth
```

**Produces:** `spec.md`, `checklists/requirements.md`, new git branch

#### `/speckit.clarify` — Resolve Ambiguities

Identifies underspecified areas in the current spec and asks up to 5 targeted questions. Answers are encoded back into `spec.md`.

```
/speckit.clarify
```

#### `/speckit.plan` — Create Implementation Plan

Generates the technical architecture from the spec: data models, API contracts, and dev environment setup.

```
/speckit.plan
```

**Produces:** `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

#### `/speckit.tasks` — Generate Task Breakdown

Creates a dependency-ordered, actionable task list from plan and spec artifacts.

```
/speckit.tasks
```

**Produces:** `tasks.md` with phased tasks, dependency graph, and parallel execution markers

#### `/speckit.analyze` — Cross-Artifact Analysis

Read-only consistency check across `spec.md`, `plan.md`, and `tasks.md`. Finds gaps, duplications, ambiguities, and constitution violations.

```
/speckit.analyze
```

#### `/speckit.implement` — Execute Tasks

Processes all tasks from `tasks.md` in order, delegating to backend and web developer agents as appropriate. Uses TDD — tests are written before implementation.

```
/speckit.implement
```

#### `/speckit.checklist` — Generate Quality Checklist

Creates requirement quality checklists that validate whether specs are complete, clear, and testable (not implementation checklists).

```
/speckit.checklist
```

#### `/speckit.constitution` — Project Governance

Creates or updates the project constitution — the non-negotiable rules governing architecture, tech stack, code quality, security, and testing.

```
/speckit.constitution
```

**Produces:** `.specify/memory/constitution.md`

#### `/speckit.taskstoissues` — Export Tasks to GitHub Issues

Converts tasks from `tasks.md` into dependency-ordered GitHub issues.

```
/speckit.taskstoissues
```

### Figma Workflow

These skills handle the Figma design integration pipeline. Located in `.claude/skills/`.

#### `/cgs.figma.capture` — Capture Screens to Figma

Captures the app's screen states from e2e tests and pushes them into a Figma file as design frames.

```
/cgs.figma.capture https://figma.com/design/abc123/file-name
/cgs.figma.capture https://figma.com/design/abc123/file-name --new
/cgs.figma.capture https://figma.com/design/abc123/file-name "Add Task Modal" --viewport=D,M
```

**Options:** `--new` (only new screens), `--viewport=D,M,T` (filter viewports), screen filter (substring match)

**Produces:** `tests/e2e/<feature>/capture-screens.spec.ts`, Figma frames

#### `/cgs.figma.link` — Link Figma to Spec

Generates the `## Screens` section in `spec.md` and auto-matches Figma frames to each screen by name similarity.

```
/cgs.figma.link https://figma.com/design/abc123/file-name
/cgs.figma.link https://figma.com/design/abc123/file-name US1,US3
/cgs.figma.link https://figma.com/design/abc123/file-name --force
```

**Options:** `--force` (overwrite existing links), story filter

**Produces:** Updated `spec.md` with `## Screens` table and `**Design**:` links on User Stories

#### `/cgs.figma.visual` — Visual Comparison

Runs pixelmatch-based visual comparison of rendered UI against Figma designs. Each screen with a Figma Node value gets a pixel-level diff test.

```
/cgs.figma.visual
/cgs.figma.visual US1
/cgs.figma.visual --threshold=0.05
```

**Options:** `--threshold=0.05` (override default 2% diff threshold), screen/story filter

**Produces:** `tests/e2e/<feature>/figma-visual.spec.ts`, `tests/e2e/helpers/figma-compare.ts`

**Pipeline:**
```
spec.md → e2e tests → /cgs.figma.capture → /cgs.figma.link → /cgs.figma.visual
```

---

## Workflow Overview

### Standard Feature Development

```
/speckit.specify "feature description"     → spec.md + branch
/speckit.clarify                           → refined spec.md
/speckit.plan                              → plan.md + contracts + data model
/speckit.tasks                             → tasks.md
/speckit.implement                         → working code + tests
```

### Design-First Development

```
/speckit.specify "feature description"                         → spec.md
/cgs.figma.link <figma-url>                        → spec.md with Figma screen links
/speckit.plan                                                  → plan.md
/speckit.tasks                                                 → tasks.md
@web-developer implement from Figma                            → UI code
/cgs.figma.visual                                  → pixel-level design comparison
```

### Figma Capture & Validation Pipeline

```
/cgs.figma.capture <figma-url>                     → capture app screens to Figma
/cgs.figma.link <figma-url>                        → link Figma frames to spec
/cgs.figma.visual                                  → pixel-level comparison report
```

---

## Real-World Examples

### Example 1: Setting Up Spec Kit on an Existing Project

You have a project with code but no Spec Kit infrastructure. Here's how to bootstrap it.

**Step 1 — Copy the setup files into your project**

```bash
# From your existing project root
cp -r /path/to/cgs-team-claude-setup/.mcp.json .
cp -r /path/to/cgs-team-claude-setup/.claude .
cp -r /path/to/cgs-team-claude-setup/.specify .
```

**Step 2 — Create the project constitution**

Start Claude Code in your project and define the rules that govern your codebase:

```
/speckit.constitution
```

Claude will ask about your tech stack, architecture principles, testing requirements, and coding standards. Answer based on your existing project. For example:

> **Claude:** What is the primary tech stack?
> **You:** React 18 + TypeScript frontend, NestJS backend, PostgreSQL with Prisma ORM, deployed on AWS

The constitution is written to `.specify/memory/constitution.md`. All future specs, plans, and implementations will follow these rules.

**Step 3 — Create specs for existing and new features**

Since this is an existing project, start by asking Claude to analyze what's already built. Point it at the source code and any existing documentation:

```
Analyze the current codebase and docs/architecture.md to identify
the main feature areas already implemented in this project.
List each feature with a short summary of what it does.
```

Claude will scan the codebase structure, routes, modules, and docs to produce a feature inventory. Use this as your starting point.

Then create specs for each feature area, referencing the existing code and docs so the spec captures what's already built:

```
/speckit.specify User authentication — already implemented in src/modules/auth/.
See docs/authentication.md for the original PRD.
Supports email/password, Google OAuth, and magic links.
```

Claude reads the referenced files, analyzes the existing implementation, and generates a `spec.md` that reflects the current state — not a greenfield design.

```
/speckit.clarify
```

Resolve any ambiguities — Claude asks up to 5 targeted questions and encodes the answers back into the spec.

```
/speckit.plan
```

Generates the technical plan: architecture, data model, API contracts, and dev environment setup — all aligned with your constitution.

```
/speckit.tasks
```

Breaks the plan into dependency-ordered, actionable tasks ready for implementation.

**Step 4 — Implement**

```
/speckit.implement
```

Claude processes tasks in order, delegating to the backend-developer and web-developer agents. Tests are written before code. Completed tasks are checked off in `tasks.md`.

**Step 5 (optional) — Capture existing screens into Figma**

If you have e2e tests, capture your app's screens into Figma. This gives your design team a baseline to work from for future redesigns:

```
/cgs.figma.capture https://figma.com/design/abc123/MyApp-Current-UI
```

Then link the Figma file to your specs:

```
/cgs.figma.link https://figma.com/design/abc123/MyApp-Current-UI
```

Now your design team can duplicate the Figma file, create redesigned versions, and developers can use `/cgs.figma.visual` to validate implementations against the new designs.

**What you get:**
- MCP servers (Playwright, Figma, Chrome DevTools) for browser automation and design integration
- 2 specialized agents (backend and web developer) that understand your project's conventions
- 12 skills for the full specify → plan → implement → Figma validation lifecycle
- A constitution that enforces consistency across all future work
- (Optional) Figma baseline for your design team to iterate on

---

### Example 2: Converting Existing Docs and Code into Specs

You have a running project with existing documentation (PRDs, READMEs, wiki pages, Confluence docs) and working source code, but no formal specs. Here's how to retroactively create specs for each feature.

**Step 1 — Set up constitution first (if not done)**

```
/speckit.constitution
```

This ensures the generated specs follow your project's actual conventions.

**Step 2 — Create a spec from existing documentation**

Feed your existing docs directly to the specify command. Claude will read them, extract requirements, and produce a structured spec:

```
/speckit.specify Based on our existing user auth system:
- See docs/authentication.md for the original PRD
- The implementation is in src/modules/auth/
- We support email/password, Google OAuth, and magic links
- JWT tokens with refresh rotation
- Rate limiting on login attempts
```

Claude will read the referenced files, analyze the existing implementation, and generate a `spec.md` that captures what's already built — not what needs to be built.

**Step 3 — Clarify gaps between docs and implementation**

Documentation often drifts from reality. Use clarify to surface the differences:

```
/speckit.clarify
```

Claude will ask targeted questions like:
> "The docs mention 2FA support but the codebase has no 2FA implementation. Should the spec reflect current state (no 2FA) or intended state (with 2FA)?"

Answer based on what you want the spec to represent.

**Step 4 — Generate the technical plan from existing code**

```
/speckit.plan
```

Since the code already exists, the plan will document the current architecture rather than propose a new one: data models, API contracts, project structure, and dev environment setup.

**Step 5 — Generate tasks for remaining work**

```
/speckit.tasks
```

If the spec captures only what exists, the task list will be minimal (mostly test coverage). If you included planned features, tasks will cover the delta between current state and target state.

**Step 6 — Validate with analysis**

```
/speckit.analyze
```

This cross-checks `spec.md`, `plan.md`, and `tasks.md` for consistency. It catches things like:
- Requirements in the spec with no corresponding tasks
- Tasks that reference contracts not defined in the plan
- Terminology drift between documents

**Step 7 (optional) — Link Figma designs to specs**

If you have Figma designs for this feature, link them to the spec:

```
/cgs.figma.link https://figma.com/design/abc123/MyDesign
```

Claude reads the Figma file, finds all frames, and auto-matches them to screens in your `spec.md`. Each User Story gets a `**Design**:` link to the relevant Figma frame. This enables `/cgs.figma.visual` pixel-level comparisons later.

**Step 8 — Implement remaining work**

With specs, plans, tasks, and (optionally) Figma links in place, execute the implementation:

```
/speckit.implement
```

Claude processes tasks from `tasks.md` in dependency order — delegating backend work to the backend-developer agent and frontend work to the web-developer agent. Tests are written before implementation (TDD). Each completed task is checked off in `tasks.md`.

If Figma screens are linked, the web-developer agent will pull design context directly from Figma to match the implementation to the designs.

**Repeat for each major feature area.** A typical project might produce specs for: authentication, user management, billing, notifications, etc. — one spec per feature branch.

**Pro tip:** For large codebases, work feature-by-feature rather than trying to spec the entire app at once. Each `/speckit.specify` creates its own branch and isolated spec directory.

---

### Example 3: Capturing App Screens into Figma

You have a running application with e2e tests but no Figma designs. You want to capture every screen into Figma so you can use the visual validation workflow and link designs to specs.

**Prerequisites:**
- E2e tests must exist for the feature (in `tests/e2e/<feature>/`)
- You need a Figma account with edit access
- Playwright must be configured with a `webServer` entry (no manual dev server needed)

**Step 1 — Capture screens into Figma**

The capture skill discovers screens from your e2e tests, generates a Playwright capture script, and pushes frames to Figma:

```
/cgs.figma.capture https://figma.com/design/abc123/My-App-Screens
```

Claude will:
1. Analyze your e2e test files to discover all screen states
2. Determine viewports per screen type (Desktop/Tablet/Mobile for fundamental states, Desktop/Mobile for modals)
3. Check which screens already exist in the Figma file
4. Present a capture plan for your approval
5. Generate `capture-screens.spec.ts` and run it

**Step 2 — Capture only new screens**

After adding new e2e tests, capture only the screens not yet in Figma:

```
/cgs.figma.capture https://figma.com/design/abc123/My-App-Screens --new
```

**Step 3 — Capture specific screens or viewports**

```
/cgs.figma.capture https://figma.com/design/abc123/My-App-Screens "Add Task Modal" --viewport=D,M
```

**Step 4 — Link Figma frames to specs**

```
/cgs.figma.link https://figma.com/design/abc123/My-App-Screens
```

Claude reads the Figma file structure, auto-matches frames to screens by name similarity, and updates `spec.md` with the `## Screens` table and `**Design**:` links on each User Story.

**Step 5 — Run visual comparison**

With screens linked, run pixel-level comparison anytime:

```
/cgs.figma.visual
```

This generates a Playwright test that compares live rendered pages against Figma frames using pixelmatch with a 2% default threshold. As the app evolves, re-capture updated screens and re-run to catch visual regressions.

**Full pipeline in action:**

```
# 1. Capture all screens into Figma from e2e tests
/cgs.figma.capture https://figma.com/design/abc123/...

# 2. Link Figma frames to spec
/cgs.figma.link https://figma.com/design/abc123/...

# 3. Pixel-level visual comparison
/cgs.figma.visual

# 4. After code changes, re-capture and re-validate
/cgs.figma.capture https://figma.com/design/abc123/... "Dashboard"
/cgs.figma.visual
```

---

## Project Structure

```
.
├── .mcp.json                          # MCP server configuration
├── .claude/
│   ├── agents/
│   │   ├── backend-developer.md       # Backend agent definition
│   │   └── web-developer.md           # Frontend agent definition
│   ├── commands/
│   │   ├── speckit.specify.md         # Feature spec generation
│   │   ├── speckit.clarify.md         # Spec clarification
│   │   ├── speckit.plan.md            # Implementation planning
│   │   ├── speckit.tasks.md           # Task breakdown
│   │   ├── speckit.analyze.md         # Cross-artifact analysis
│   │   ├── speckit.implement.md       # Task execution
│   │   ├── speckit.checklist.md       # Quality checklists
│   │   ├── speckit.constitution.md    # Project governance
│   │   └── speckit.taskstoissues.md   # GitHub issue export
│   ├── skills/
│   │   ├── cgs.figma.capture/  # Capture app screens to Figma
│   │   ├── cgs.figma.link/     # Link Figma frames to spec
│   │   └── cgs.figma.visual/   # Visual comparison against Figma
│   └── settings.local.json            # Local permissions
├── .specify/
│   ├── memory/
│   │   └── constitution.md            # Project constitution
│   ├── templates/
│   │   ├── spec-template.md           # Spec scaffold
│   │   ├── plan-template.md           # Plan scaffold
│   │   ├── tasks-template.md          # Tasks scaffold
│   │   ├── checklist-template.md      # Checklist scaffold
│   │   ├── constitution-template.md   # Constitution scaffold
│   │   └── agent-file-template.md     # Agent config scaffold
│   └── scripts/bash/
│       ├── create-new-feature.sh      # Feature branch + spec init
│       ├── check-prerequisites.sh     # Feature context discovery
│       ├── setup-plan.sh              # Plan initialization
│       ├── update-agent-context.sh    # Agent context refresh
│       └── common.sh                  # Shared utilities
└── README.md
```
