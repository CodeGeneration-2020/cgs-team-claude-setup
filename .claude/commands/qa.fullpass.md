---
description: Run a full QA pass on the current feature — manual tests, e2e tests, design comparison, and bug reports.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Perform a complete QA validation of the current feature by delegating to the **qa-agent**. This is the most comprehensive testing mode — it covers codebase analysis, spec analysis, manual browser testing, Figma design comparison, e2e test writing, and bug reporting.

## Execution

1. **Determine scope**: If the user provided arguments (e.g., a specific feature name or spec path), use that as context. Otherwise, the QA agent will auto-discover the active feature from the current git branch.

2. **Launch the QA agent** using the Task tool with `subagent_type: "qa-agent"`. Pass this prompt:

   ```
   Run a full QA pass on the current feature.

   Follow Mode 1 (Full QA Pass) exactly:
   1. Bootstrap — discover project context (constitution, spec, plan, quickstart)
   2. Read spec — extract all User Stories, acceptance scenarios, edge cases, and Figma Screen links
   3. Analyze codebase — scan relevant app(s) for structural issues against constitution
   4. Build test plan — map each acceptance scenario to concrete test steps
   5. For each User Story (in priority order P1 first):
      a. If Figma Screens exist: run design comparison
      b. Run manual browser tests for each acceptance scenario
      c. Check console errors and network requests
      d. File bug reports for any failures
   6. Write e2e tests for all acceptance scenarios and edge cases
   7. Run e2e tests and capture results
   8. Produce summary report at reports/qa/YYYY-MM-DD-summary.md

   Additional user context: <user arguments here, or "none">
   ```

3. **Report results**: When the QA agent completes, summarize the findings back to the user:
   - Number of User Stories tested
   - Pass/fail counts
   - Number of bugs filed (with file paths)
   - E2E tests written (with file paths)
   - Overall assessment
   - Path to the summary report
