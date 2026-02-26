---
description:
  Enrich the feature specification with Figma screen links by auto-matching design screens to User
  Stories.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

Goal: Parse a Figma design file URL, discover all screens (top-level frames within pages),
auto-match each screen to the most relevant User Story(ies) in the feature spec, and write Figma
screen links directly into the spec under each matched User Story.

### 1. Parse Figma URL

Extract the Figma `fileKey` (and optionally `nodeId`) from the user-provided URL in `$ARGUMENTS`.

**URL formats to support**:

- `https://figma.com/design/{fileKey}/{fileName}` — full file
- `https://figma.com/design/{fileKey}/{fileName}?node-id={nodeId}` — specific node
- `https://www.figma.com/design/{fileKey}/{fileName}?node-id={int1}-{int2}` — with www prefix
- `https://figma.com/design/{fileKey}/branch/{branchKey}/{fileName}` — branch URL (use `branchKey`
  as fileKey)

**Extraction rules**:

- `fileKey`: the path segment after `/design/` (or use `branchKey` if branch URL format)
- `nodeId`: from query parameter `node-id`, convert `-` to `:` for MCP API calls (e.g., `1-234`
  becomes `1:234`)
- If no URL is provided or cannot be parsed: ERROR "No valid Figma URL provided. Usage:
  `/speckit.figmalink https://figma.com/design/{fileKey}/{fileName}`"

### 2. Load Feature Context

Run `.specify/scripts/bash/check-prerequisites.sh --json --paths-only` from repo root **once**.
Parse the JSON output for:

- `FEATURE_DIR`
- `FEATURE_SPEC`

For single quotes in args, use escape syntax: e.g `'I'\''m Groot'` (or double-quote if possible:
`"I'm Groot"`).

If `FEATURE_SPEC` does not exist or cannot be found: ERROR "Spec file not found. Run
`/speckit.specify` first."

Read the spec file at `FEATURE_SPEC`.

### 3. Parse User Stories from Spec

Extract all User Stories from the spec. Each User Story follows this heading pattern:

```
### User Story {N} - {Title} (Priority: P{N})
```

For each User Story, capture:

- **Number** (e.g., `1`, `2`, ... `12`)
- **Title** (e.g., `User Registration & Access Management`)
- **Priority** (e.g., `P1`)
- **Full heading line** (for precise location matching when writing back)
- **Description paragraph** (the text between the heading and the first bold marker)
- **Acceptance Scenarios** (the numbered Given/When/Then items)
- **Whether a `**Figma Screens**:` section already exists** (for idempotency)

### 4. Fetch Figma File Structure

Use the Figma MCP `get_metadata` tool to retrieve the file structure.

**Strategy for multi-page files**:

a. First, call `get_metadata` with `nodeId: "0:1"` (the root page node) and the extracted `fileKey`
to get the top-level structure including all pages.

b. The returned XML will contain page elements, each with child frame elements. Parse this to
identify:

- **Pages**: Top-level containers (e.g., "Login Flows", "Dashboard", "Procurement")
- **Screens**: Top-level frames within each page — these are the design screens

c. If the initial `get_metadata` call only returns page-level nodes without frame children, make
additional `get_metadata` calls for each page node to get the frames within them.

d. For each identified screen, capture:

- **nodeId** (in `X:Y` format for API, convert to `X-Y` for URL links)
- **name** (the frame/artboard name as set by the designer)
- **parent page name** (for additional context during matching)

**Screen detection heuristic**: Top-level frames within a page are screens. Nested frames,
components, groups, and other non-frame layer types at the top level should be excluded. If a page
contains only a single large frame, treat it as a screen. If a page has no frames (only loose
elements), skip that page.

**Edge case — nodeId provided in URL**: If the user provided a specific `nodeId` in the URL, use
that as the starting point instead of `0:1`. This allows targeting a specific page or section of the
Figma file.

### 5. Auto-Match Screens to User Stories

For each screen identified in step 4, determine which User Story(ies) it relates to using semantic
reasoning. Consider these matching signals:

**Primary signals** (strong match indicators):

- Screen name contains keywords directly from the User Story title (e.g., screen "Login Page"
  matches "User Registration & Access Management")
- Screen name describes an action or view mentioned in acceptance scenarios (e.g., screen "RFQ
  Comparison View" matches "Quote Review & Approval")
- Screen name matches a Key Entity concept from the story (e.g., screen "Vendor Profile" matches
  "Vendor & Supplier Management")

**Secondary signals** (supporting context):

- Page name in Figma provides domain grouping (e.g., screens on a page named "Invoicing" likely
  match "Invoice Reconciliation & Dispute Resolution")
- Functional workflow adjacency (e.g., "OTP Verification" screen relates to "User Registration &
  Access Management" even if the name doesn't directly mention registration)

**Matching rules**:

- A screen CAN match multiple User Stories (e.g., a "Navigation Bar" screen might relate to multiple
  stories)
- A User Story CAN have zero matched screens (acceptable — not all stories have dedicated screens)
- A User Story CAN have multiple matched screens
- Prefer precision over recall — only match when there is clear semantic relevance
- If a screen name is too generic (e.g., "Frame 1", "Untitled") and cannot be meaningfully matched,
  skip it
- Group related screens together when they clearly form a flow (e.g., "Login Step 1", "Login Step
  2", "Login OTP" all match the same User Story)

**Output**: Produce an internal mapping:

```
User Story N -> [
  { screenName: "Screen A", nodeId: "X:Y", pageName: "Page Z" },
  { screenName: "Screen B", nodeId: "X:Y", pageName: "Page Z" },
  ...
]
```

Also track unmatched screens (screens that did not match any story) for the completion report.

### 6. Write Figma Links into Spec

For each User Story that has matched screens, insert (or update) a `**Figma Screens**:` section
immediately after the User Story heading line and before the description paragraph.

**Insertion format**:

```markdown
### User Story {N} - {Title} (Priority: P{N})

**Figma Screens**:

- [{Screen Name}](https://figma.com/design/{fileKey}/?node-id={nodeId-with-dashes})
- [{Screen Name 2}](https://figma.com/design/{fileKey}/?node-id={nodeId-with-dashes})

{Existing description paragraph...}
```

**Formatting rules**:

- Node ID in URL uses dash separator (e.g., `1-234`), not colon
- Screen names should be used as-is from Figma (preserving the designer's naming)
- Screens should be listed in logical order: group by page, then by position (top-to-bottom,
  left-to-right)
- Each link is a markdown list item with the screen name as link text
- One blank line after the last screen link before the existing description paragraph
- One blank line between the heading and the `**Figma Screens**:` label

**Idempotency handling**:

- If a `**Figma Screens**:` section already exists under a User Story (detected in step 3):
  - Replace the entire existing `**Figma Screens**:` block (from the `**Figma Screens**:` line
    through all consecutive `- [` list items) with the new set of links
  - This ensures re-running the skill updates rather than duplicates
- If no existing section: insert the new block

**Edge case — no matches for a story**: Do not insert an empty `**Figma Screens**:` section. Simply
leave the User Story unchanged.

Write the updated spec back to `FEATURE_SPEC`.

### 7. Report Completion

Output a summary report with:

1. **Figma file info**: File key, number of pages scanned, total screens discovered
2. **Matching summary table**:

| User Story  | Screens Matched | Screen Names                 |
| ----------- | --------------- | ---------------------------- |
| US1 - Title | 3               | Screen A, Screen B, Screen C |
| US2 - Title | 2               | Screen D, Screen E           |
| US3 - Title | 0               | —                            |

3. **Unmatched screens**: Screens that did not match any story (with node IDs for manual review)
4. **Spec file path**: Full path to the updated spec
5. **Stories with no screens**: List story numbers/titles that got zero matches

## Error Handling

- **Invalid URL**: Show usage example and abort
- **Figma API failure**: If `get_metadata` fails (permissions, invalid fileKey), report the error
  clearly and suggest verifying the URL and Figma access
- **No screens found**: If the Figma file contains no identifiable screens (top-level frames),
  report this and abort without modifying the spec
- **Empty spec / no User Stories**: If no User Stories are found in the spec, ERROR with suggestion
  to run `/speckit.specify` first
- **All screens unmatched**: If no screen matches any User Story, report this clearly but do not
  modify the spec; list all screens for manual review

## Key Rules

- **Fully automatic**: Never ask for user confirmation — auto-match and write directly
- **Non-destructive**: Never remove existing spec content other than replacing a prior
  `**Figma Screens**:` block
- **Idempotent**: Safe to re-run — updates existing blocks instead of duplicating
- **Preserve formatting**: Keep all existing spec formatting, heading hierarchy, and content
  ordering intact
- **Only touch User Stories**: Do not modify any section other than inserting/updating
  `**Figma Screens**:` blocks
- Use the Figma MCP `get_metadata` tool (not `get_screenshot` or `get_design_context`) for structure
  discovery
- The Figma link URL format MUST be: `https://figma.com/design/{fileKey}/?node-id={nodeId}` where
  nodeId uses `-` separator
