# CGS Team Claude Setup

A complete AI-assisted development workflow powered by [Claude Code](https://claude.com/claude-code), [Spec Kit](https://github.com/lsendel/spec-kit-mcp), and the Model Context Protocol (MCP).

This repo provides the full infrastructure for spec-driven development: from writing feature specs, through planning and implementation, to QA and visual validation against Figma designs.

---

## Table of Contents

- [Quick Start](#quick-start)
- [MCP Servers](#mcp-servers)
- [Agents](#agents)
- [Skills / Commands](#skills--commands)
  - [Specification Workflow](#specification-workflow)
  - [QA Workflow](#qa-workflow)
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

### QA Agent

| | |
|---|---|
| **File** | `.claude/agents/qa-agent.md` |
| **Focus** | Testing and validation — e2e tests, manual browser testing, visual audits, bug reports |
| **Tools** | Read, Write, Edit, Bash, Grep, Glob, Task + Figma MCP + Playwright MCP + Chrome DevTools MCP |
| **Capabilities** | Codebase analysis, spec-based test plans, Playwright e2e tests, manual browser testing, Figma design comparison, structured bug reports |

The QA agent is typically invoked through `/qa.*` skills rather than directly.

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

#### `/speckit.commit` — Logical Git Commits

Creates logical, feature-grouped git commits from uncommitted changes. Runs lint/build pre-flight checks, groups changes by User Story, updates documentation incrementally per commit, and enforces a docs-first staging rule.

```
/speckit.commit
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

#### `/speckit.figmalink` — Link Figma Designs to Spec

Auto-matches Figma screens to User Stories in `spec.md` by analyzing screen names and story content.

```
/speckit.figmalink https://figma.com/design/abc123/MyDesign
```

#### `/speckit.taskstoissues` — Export Tasks to GitHub Issues

Converts tasks from `tasks.md` into dependency-ordered GitHub issues.

```
/speckit.taskstoissues
```

### QA Workflow

#### `/qa.fullpass` — Full QA Pass

Comprehensive feature validation: manual browser testing, e2e tests, visual design comparison, and bug reports for every User Story.

```
/qa.fullpass
```

**Produces:** `reports/qa/YYYY-MM-DD-summary.md`, bug reports, e2e test files

#### `/qa.e2e` — Write & Run E2E Tests

Generates Playwright e2e tests from spec acceptance scenarios and edge cases, then runs them.

```
/qa.e2e
```

**Produces:** `tests/e2e/<feature>/*.spec.ts`

#### `/qa.targeted` — Targeted Testing

Tests a specific User Story, route, or flow instead of the full feature.

```
/qa.targeted US3 - User Login Flow
```

#### `/qa.visual` — Visual Audit

Compares rendered UI against Figma designs. Classifies discrepancies by type (layout, colors, typography, components, states) and severity.

```
/qa.visual
```

#### `/qa.codebase` — Codebase Analysis

Static analysis validating project structure and patterns against the constitution. No browser testing — pure code inspection.

```
/qa.codebase
```

#### `/qa.bugreport` — File Bug Report

Creates a structured bug report with screenshots, console errors, network request logs, and spec traceability.

```
/qa.bugreport Login button does nothing when clicked on mobile viewport
```

**Produces:** `reports/qa/YYYY-MM-DD-HHmm-<slug>.md`

---

## Workflow Overview

### Standard Feature Development

```
/speckit.specify "feature description"     → spec.md + branch
/speckit.clarify                           → refined spec.md
/speckit.plan                              → plan.md + contracts + data model
/speckit.tasks                             → tasks.md
/speckit.implement                         → working code + tests
/speckit.commit                            → logical feature-grouped commits
/qa.fullpass                               → test results + bug reports
```

### Design-First Development

```
/speckit.specify "feature description"     → spec.md
/speckit.figmalink <figma-url>             → spec.md with Figma screen links
/speckit.plan                              → plan.md
/speckit.tasks                             → tasks.md
@web-developer implement from Figma        → UI code
/qa.visual                                 → design comparison report
```

### QA-Only Pass

```
/qa.codebase                               → structural analysis
/qa.e2e                                    → e2e test suite
/qa.visual                                 → Figma comparison
/qa.fullpass                               → everything above + manual testing
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

**Step 3 — Verify the setup**

```
/qa.codebase
```

This runs a static analysis of your existing code against the new constitution. It will flag any violations — treat these as a backlog, not blockers.

**Step 4 — Create specs for existing and new features**

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

**Step 5 — Implement**

```
/speckit.implement
```

Claude processes tasks in order, delegating to the backend-developer and web-developer agents. Tests are written before code. Completed tasks are checked off in `tasks.md`.

**Step 6 — QA**

```
/qa.fullpass
```

Runs manual browser testing, e2e tests, and files bug reports for anything that doesn't match the spec.

**Step 7 (optional) — Capture existing screens into Figma**

If you don't have Figma designs yet, capture your running app's screens into Figma. This gives your design team a baseline to work from for future redesigns:

```
Capture http://localhost:3000/dashboard into Figma as a new file called "MyApp - Current UI"
```

Then capture the remaining screens into the same file:

```
Capture these pages into the Figma file https://figma.com/design/abc123/MyApp-Current-UI:
- http://localhost:3000/login
- http://localhost:3000/settings
- http://localhost:3000/users
```

Once captured, link the Figma file to your specs:

```
/speckit.figmalink https://figma.com/design/abc123/MyApp-Current-UI
```

Now your design team can duplicate the Figma file, create redesigned versions, and developers can use `/qa.visual` to validate implementations against the new designs.

**What you get:**
- MCP servers (Playwright, Figma, Chrome DevTools) for browser testing and design integration
- 3 specialized agents that understand your project's conventions
- 17 skills for the full specify → plan → implement → QA lifecycle
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

If you have Figma designs for this feature, link them to the spec so agents and QA can reference them:

```
/speckit.figmalink https://figma.com/design/abc123/MyDesign
```

Claude reads the Figma file, finds all screens, and auto-matches them to User Stories in your `spec.md`. Each story gets a `**Figma Screens:**` section with direct links to the relevant design frames. This enables `/qa.visual` comparisons later.

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

You have a running application but no Figma designs. You want to capture every screen into Figma so you can use the visual QA workflow (`/qa.visual`) and link designs to specs (`/speckit.figmalink`).

**Prerequisites:**
- Your app must be running locally (e.g., `http://localhost:3000`)
- You need a Figma account with edit access

**Step 1 — Start your app**

```bash
# In a separate terminal
npm run dev
```

**Step 2 — Capture screens into a new Figma file**

In Claude Code, ask to capture your first screen:

```
Capture http://localhost:3000/dashboard into Figma as a new file called "My App Screens"
```

Claude will use the Figma MCP's `generate_figma_design` tool to:
1. Open the URL in a headless browser
2. Capture the rendered page
3. Create a new Figma file with the captured design

You'll receive a Figma URL. Open it to claim the file.

**Step 3 — Capture remaining screens into the same file**

Once you have the file, add more screens to it:

```
Capture these pages into the existing Figma file https://figma.com/design/abc123/My-App-Screens:
- http://localhost:3000/login
- http://localhost:3000/signup
- http://localhost:3000/settings
- http://localhost:3000/users
- http://localhost:3000/users/123
```

Each page is captured and added as a separate frame in the Figma file. Name each frame to match the route or feature (e.g., "Login", "Signup", "User Detail").

**Step 4 — Capture responsive variants**

For responsive designs, capture at different viewports. Ask Claude to resize before capturing:

```
Capture http://localhost:3000/dashboard at mobile (375x812) and tablet (768x1024)
viewport sizes into the Figma file https://figma.com/design/abc123/My-App-Screens
```

**Step 5 — Capture interactive states**

For pages with multiple states (empty, loading, error, populated), capture each:

```
Capture these states of the users page into Figma:
1. http://localhost:3000/users (populated list)
2. Navigate to users, clear all data, capture the empty state
3. Disconnect the API and capture the error state
```

Claude will use Playwright MCP to navigate, interact with the page, and Chrome DevTools MCP to manipulate network conditions before each capture.

**Step 6 — Link Figma screens to specs**

Now that you have Figma designs, link them to your feature specs:

```
/speckit.figmalink https://figma.com/design/abc123/My-App-Screens
```

Claude reads the Figma file structure, finds all frames/screens, and auto-matches them to User Stories in your `spec.md` by comparing screen names to story content. The spec is updated with `**Figma Screens:**` links under each matching User Story.

**Step 7 — Run visual QA**

With screens linked, you can now run visual audits anytime:

```
/qa.visual
```

This compares the live app against the Figma captures, classifying discrepancies by type (layout, colors, typography) and severity. As the app evolves, re-capture updated screens and re-run the audit to catch visual regressions.

**Full pipeline in action:**

```
# 1. Capture all screens into Figma
"Capture http://localhost:3000/login into a new Figma file"
"Capture /dashboard, /settings, /users into the same file"

# 2. Link to specs
/speckit.figmalink https://figma.com/design/abc123/...

# 3. Visual QA against captures
/qa.visual

# 4. After code changes, re-capture and re-validate
"Re-capture http://localhost:3000/dashboard into Figma"
/qa.visual
```

---

## Project Structure

```
.
├── .mcp.json                          # MCP server configuration
├── .claude/
│   ├── agents/
│   │   ├── backend-developer.md       # Backend agent definition
│   │   ├── web-developer.md           # Frontend agent definition
│   │   └── qa-agent.md                # QA agent definition
│   ├── commands/
│   │   ├── speckit.specify.md         # Feature spec generation
│   │   ├── speckit.clarify.md         # Spec clarification
│   │   ├── speckit.plan.md            # Implementation planning
│   │   ├── speckit.tasks.md           # Task breakdown
│   │   ├── speckit.analyze.md         # Cross-artifact analysis
│   │   ├── speckit.implement.md       # Task execution
│   │   ├── speckit.commit.md          # Logical git commits
│   │   ├── speckit.checklist.md       # Quality checklists
│   │   ├── speckit.constitution.md    # Project governance
│   │   ├── speckit.figmalink.md       # Figma-to-spec linking
│   │   ├── speckit.taskstoissues.md   # GitHub issue export
│   │   ├── qa.fullpass.md             # Full QA pass
│   │   ├── qa.e2e.md                  # E2E test generation
│   │   ├── qa.targeted.md             # Targeted testing
│   │   ├── qa.visual.md               # Visual audit
│   │   ├── qa.codebase.md            # Codebase analysis
│   │   └── qa.bugreport.md            # Bug reporting
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
