---
description: Run a visual audit comparing the rendered UI against Figma designs for all or specific screens.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Compare the implemented UI against the original Figma designs. This captures screenshots of both the live app and the Figma mockups, then classifies discrepancies by severity (Critical, Major, Minor, Cosmetic).

## Execution

1. **Determine scope**: If the user provided arguments (e.g., a specific User Story, screen name, or Figma URL), scope the audit to that. Otherwise, audit all screens that have `**Figma Screens**:` links in the spec.

2. **Launch the QA agent** using the Task tool with `subagent_type: "qa-agent"`. Pass this prompt:

   ```
   Run a visual audit (Mode 3).

   Follow Mode 3 (Visual Audit) exactly:
   1. Bootstrap — discover project context (constitution, spec, plan, quickstart)
   2. Read spec to find all User Stories with **Figma Screens**: sections
   3. For each screen link:
      a. Navigate to the corresponding app route
      b. Screenshot the rendered page via Playwright MCP
      c. Screenshot the Figma design via Figma MCP get_screenshot
      d. Compare and classify discrepancies:
         - Layout: Element positioning, alignment, spacing
         - Colors: Background, text, border, button colors
         - Typography: Font sizes, weights, line heights
         - Components: Correct component types
         - States: Hover, active, disabled, error states
         - Icons: Correct icons in correct positions
      e. Classify severity: Critical, Major, Minor, Cosmetic
   4. Produce a visual audit report listing all discrepancies by severity

   Scope: <user arguments or "all screens with Figma links in spec">
   ```

3. **Report results**: When the QA agent completes, summarize:
   - Number of screens compared
   - Discrepancy counts by severity
   - Worst offenders (Critical/Major issues)
   - Path to the audit report
