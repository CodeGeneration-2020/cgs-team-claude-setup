---
description: Write and run Playwright e2e tests for the current feature based on spec acceptance scenarios and edge cases.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Generate executable Playwright e2e tests that cover all acceptance scenarios and edge cases from the feature spec. Optionally scope to specific User Stories.

## Execution

1. **Determine scope**: If the user provided arguments (e.g., `US1`, `US3`, `only edge cases`), scope test generation to those. Otherwise, generate tests for all User Stories.

2. **Launch the QA agent** using the Task tool with `subagent_type: "qa-agent"`. Pass this prompt:

   ```
   Write and run Playwright e2e tests (Capability 3: E2E Test Writing).

   Steps:
   1. Bootstrap — discover project context (constitution, spec, plan, quickstart)
   2. Read spec to extract all User Stories, acceptance scenarios, and edge cases
   3. For each User Story (scoped to user's request if specified):
      a. Create test file at tests/e2e/<feature>/us<NN>-<slug>.spec.ts
      b. Write tests following the standard structure:
         - test.describe('US{N} - <Title>')
         - test('AS{N}: <scenario summary>') for each acceptance scenario
         - test('Edge: <description>') for each edge case
         - Use Given/When/Then comments from the spec
         - Discover app URLs from quickstart.md
      c. Cover all acceptance scenarios AND edge cases
   4. Run all written e2e tests and capture results
   5. Report which tests pass and which fail

   Scope: <user arguments or "all User Stories">
   ```

3. **Report results**: When the QA agent completes, summarize:
   - Test files created (with paths)
   - Number of tests per User Story
   - Pass/fail results from the test run
   - Any failing tests that indicate bugs
