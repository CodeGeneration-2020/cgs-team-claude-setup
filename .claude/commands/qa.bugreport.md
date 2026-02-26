---
description: File a structured bug report for a specific issue, with screenshots and spec traceability.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding. The user must describe the bug or issue they want reported. If the input is empty, ask: "Describe the bug you want me to investigate and report. Include what you observed, what you expected, and which page/flow it affects."

## Goal

Investigate a specific issue in the running application and file a structured bug report in `reports/qa/` with screenshots, console errors, network request details, and traceability to the spec's User Story and acceptance scenario.

## Execution

1. **Validate input**: The user must describe the bug. Accepted formats:
   - A bug description: `login button does nothing when clicked`
   - A reference + issue: `US2 - form validation is missing for email field`
   - A route + problem: `/settings page crashes when saving`

   If the input is empty, ask the user for a description before proceeding.

2. **Launch the QA agent** using the Task tool with `subagent_type: "qa-agent"`. Pass this prompt:

   ```
   Investigate and file a bug report (Capability 6: Bug Reporting).

   Steps:
   1. Bootstrap — discover project context (constitution, spec, plan, quickstart)
   2. Read the spec to identify which User Story and acceptance scenario this bug relates to
   3. Navigate to the affected page in the browser via Playwright MCP
   4. Reproduce the issue step by step
   5. Capture evidence:
      a. Take a screenshot via Playwright MCP browser_take_screenshot
      b. Check console errors via Chrome DevTools MCP list_console_messages
      c. Check network requests via Chrome DevTools MCP list_network_requests
      d. Inspect specific failed requests if any (get_network_request)
   6. Create the bug report file at reports/qa/YYYY-MM-DD-HHmm-<slug>.md using the standard template:
      - Title, Date, Severity, User Story reference
      - Description, Steps to Reproduce, Expected vs Actual Behavior
      - Screenshots, Console Errors, Failed Network Requests
      - Environment details
   7. Save screenshot evidence to reports/qa/screenshots/

   Bug to investigate: <user arguments>
   ```

3. **Report results**: When the QA agent completes, provide:
   - Path to the bug report file
   - Severity assigned
   - Which User Story it's linked to
   - Brief summary of findings
