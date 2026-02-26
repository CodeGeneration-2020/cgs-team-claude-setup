---
description: Run a codebase analysis validating project structure and patterns against the constitution.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Scan the implemented codebase for structural issues by validating against the project constitution. This does NOT involve browser testing — it is a static analysis of code structure, patterns, and conventions.

## Execution

1. **Determine scope**: If the user provided arguments (e.g., a specific app name, module, or directory), scope the analysis. Otherwise, analyze all apps relevant to the current feature.

2. **Launch the QA agent** using the Task tool with `subagent_type: "qa-agent"`. Pass this prompt:

   ```
   Run a codebase analysis (Capability 1: Codebase Analysis).

   Steps:
   1. Bootstrap — load constitution from .specify/memory/constitution.md
   2. Load feature context — run check-prerequisites.sh, read plan.md for project structure and tech stack
   3. Scan the codebase and validate against constitution rules:
      - Folder structure matches what constitution and plan prescribe
      - Route definitions use proper code splitting (if required)
      - Components follow constitution patterns (e.g., no business logic in UI)
      - Error boundaries, loading states, empty states are present where needed
      - API endpoints in services match contracts in contracts/ directory
      - Validation schemas exist for all forms (if required)
      - Shared packages are used correctly (no bypassing API client, no raw HTTP calls)
   4. Report findings as a structured list with severity levels
   5. Do NOT modify any code — this is read-only analysis

   Scope: <user arguments or "all apps in the current feature">
   ```

3. **Report results**: When the QA agent completes, summarize:
   - Number of issues found by severity (Critical, Major, Minor)
   - Key violations and which constitution rules they break
   - Recommendations for fixing the top issues
