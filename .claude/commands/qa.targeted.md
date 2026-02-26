---
description: Run targeted QA testing on a specific user story, flow, or page route.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding. If the user did not specify a target (e.g., "US3", "login flow", "/dashboard"), ask them what they want to test before launching the agent.

## Goal

Test a specific user story, flow, or page — not the entire feature. This is faster than a full QA pass and is useful for validating individual pieces of functionality.

## Execution

1. **Validate input**: The user must specify what to test. Accepted formats:
   - A User Story reference: `US3`, `User Story 3`, `the registration flow`
   - A page or route: `/login`, `/dashboard`, `the settings page`
   - A flow description: `login flow`, `file upload`, `checkout process`

   If the input is empty, ask the user: "What would you like me to test? Provide a User Story (e.g., US3), a route (e.g., /login), or a flow description."

2. **Launch the QA agent** using the Task tool with `subagent_type: "qa-agent"`. Pass this prompt:

   ```
   Run targeted testing (Mode 2) on: <user's target>

   Follow Mode 2 (Targeted Testing) exactly:
   1. Bootstrap — discover project context (constitution, spec, plan, quickstart)
   2. Read only the referenced User Story from the spec
   3. Run manual browser tests for that flow only
   4. Check design comparison if Figma Screens exist for that story
   5. File bug reports for any issues found
   6. Report results for that specific story

   Target: <user arguments>
   ```

3. **Report results**: When the QA agent completes, summarize:
   - Which User Story / flow was tested
   - Pass/fail for each acceptance scenario
   - Any bugs filed (with file paths)
   - Design comparison results (if applicable)
