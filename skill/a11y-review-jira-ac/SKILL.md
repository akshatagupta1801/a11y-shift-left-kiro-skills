---
name: a11y-review-jira-ac
description: >
  Reviews code changes against the accessibility acceptance criteria in a Jira a11y subtask.
  Validates each AC against the actual code, flags missing implementations with copy-paste
  fixes, and catches new elements added since the ACs were written. Use after coding is
  complete, before raising a PR. Trigger phrases: review my a11y changes, check accessibility,
  validate a11y, did I cover all a11y requirements, review a11y for TICKET-ID.
compatibility: >-
  Requires Atlassian MCP server (atlassian_test_connection, atlassian_manifest, atlassian_read)
  to fetch the a11y subtask. Optional: a11y MCP server (a11y_read, a11y_search) for WCAG
  technique lookups when providing fixes.
metadata:
  tags:
    - a11y
    - accessibility
    - review
    - jira
    - shift-left
    - wcag
    - code-review
  version: "1.0.0"
  author: akshatagupta1801
  source: community
  lifecycle:
    - review
  mcp-prerequisites:
    - name: atlassian
      tools:
        - atlassian_test_connection
        - atlassian_manifest
        - atlassian_read
    - name: a11y
      optional: true
      tools:
        - a11y_read
        - a11y_search
---

# Review A11y Jira AC

Review code changes against the accessibility subtask created by
`a11y-story-ac-generator`. Validates each AC against the actual code,
provides fixes for gaps, and catches small drift (new elements added since
the ACs were written).

This is the **development-phase** skill in the shift-left accessibility
strategy. It runs after the developer finishes coding and before they raise
a PR.

## Prerequisites

Before proceeding, verify the required MCP servers are connected:

1. **Required:** Atlassian MCP server
   - **Verify:** Run `atlassian_test_connection()` — if Jira shows configured, proceed.

2. **Optional:** a11y MCP server (for WCAG technique lookups when providing fixes)
   - **Verify:** Run `a11y_manifest()` — if it returns operations, available.

## When to Use This Skill

Use this skill when:
- Developer says "review my a11y changes", "check accessibility", "validate a11y"
- Developer has finished coding and wants to verify against the a11y subtask
- Developer provides a Jira subtask ID or parent story ID with an a11y subtask
- Developer asks "did I cover all the a11y requirements?"

Do NOT use this skill when:
- The user wants to generate a11y ACs for a new story → use `a11y-story-ac-generator`
- The user wants a full WCAG audit → use a WCAG audit skill
- The user wants to analyze a single component in isolation

## Workflow

### Step 1: Get the A11y Subtask

Ask the developer for:
- The Jira subtask ID (e.g., `PROJ-5678`) — the a11y subtask created by `a11y-story-ac-generator`
- OR the parent story ID (e.g., `PROJ-1234`) — agent will find the a11y subtask

**If parent story provided**, find the a11y subtask:

```
atlassian_read(operationId: "j_getIssue", params: {
  issueIdOrKey: "<PARENT_STORY_ID>",
  fields: "subtasks"
})
```

Look for a subtask with "Accessibility" or "a11y" in the summary.

**Fetch the subtask description** (contains the ACs):

```
atlassian_read(operationId: "j_getIssue", params: {
  issueIdOrKey: "<SUBTASK_ID>",
  fields: "summary,description,status"
})
```

Parse the description to extract the checklist items. Each `- [ ]` or `* ( )`
line is an AC to validate.

### Step 2: Get the Code

Determine what code to review. Priority order:

1. **Developer specifies files** — "review these files: ComponentX.tsx, Modal.tsx"
2. **Git diff** — `git diff main...HEAD` or `git diff --staged` for changed files
3. **Ask** — "Which files contain the UI changes for this story?"

Read the relevant files. Focus on:
- `.tsx` / `.jsx` files (component markup and ARIA)
- `.ts` / `.js` files (keyboard handlers, focus management)
- `.css` / `.scss` / styled-components (focus indicators, contrast)

### Step 3: Analyze Screenshot (if provided)

If the developer shares a screenshot of the built UI:

1. Compare against what the ACs describe
2. Look for **new additions** not covered by the original ACs:
   - New buttons or links without expected ARIA
   - New sections or content areas
   - Tooltips, badges, or icons not in the original design
   - Status indicators or loading states
3. Note these as "⚠️ New" items in the report

If no screenshot provided, skip this step and review code only.

### Step 4: Validate Each AC Against Code

Go through each AC from the subtask one by one.

**Check if implemented:**
- Search the code for the relevant pattern (aria attribute, role, keyboard handler)
- Verify it's correct — not just present but properly wired

**Classify each AC:**
- **✅ Handled** — AC is satisfied. Briefly note where in the code.
- **❌ Missing** — AC is not implemented. Provide the fix.
- **⚠️ New** — Something in the screenshot/code needs a11y but wasn't in the original ACs.
- **ℹ️ Needs Manual Testing** — Can't verify from code alone (focus order, screen reader announcements, runtime behavior).

### Step 5: Provide Fixes for Missing Items

For each ❌ Missing item, provide:

1. **What's wrong** — one sentence
2. **File and line** — where to fix
3. **Code fix** — copy-paste ready snippet

Example:
```
❌ Chevron button missing aria-expanded

File: src/components/ObjectiveRow.tsx (Line 42)

Before:
  <button className="chevron" onClick={toggle}>
    <ChevronIcon />
  </button>

After:
  <button
    className="chevron"
    onClick={toggle}
    aria-expanded={isExpanded}
    aria-label={`${isExpanded ? 'Collapse' : 'Expand'} ${objectiveName}`}
  >
    <ChevronIcon />
  </button>
```

If the a11y MCP is available, look up the relevant WCAG technique:

```
a11y_search(operation: "search_success_criteria", params: { query: "expand collapse button" })
```

### Step 6: Report Small Drift (⚠️ New Items)

For items found from screenshot or code that weren't in the original ACs:

```
⚠️ New: AI disclaimer text at bottom needs accessible role

  Not in original ACs — found in screenshot.

  File: src/components/SuggestionsModal.tsx (Line 128)

  Fix: Add role="note" to the disclaimer paragraph:
    <p role="note" className="disclaimer">
      AI can make mistakes. Check for accuracy.
    </p>
```

### Step 7: Generate Summary Report

```
## A11y Review — <SUBTASK_ID>

### ✅ Handled
- [x] <AC text> — verified in <file>:<line>

### ❌ Missing (with fixes)
- [ ] <AC text>
  File: <path>:<line>
  Fix: <code snippet>

### ⚠️ New (not in original ACs)
- [ ] <description>
  File: <path>:<line>
  Fix: <code snippet>

### ℹ️ Needs Manual Testing
- <what to test> — <how to verify>

---

### Summary
- Handled: X / Y ACs
- Missing: X (fixes provided above)
- New issues: X
- Manual testing needed: X

### Verdict: READY FOR PR / NEEDS FIXES
```

**Verdict logic:**
- **READY FOR PR** — All ACs handled, no missing items, new items are minor
- **NEEDS FIXES** — Any ❌ Missing items exist

### Step 8: Help Developer Fix (Interactive)

After presenting the report, the developer may ask:
- "Fix the missing items for me" → Apply the code fixes
- "Show me how to fix X" → Provide detailed explanation for that item
- "Re-review after I fix" → Re-run the validation on updated code
- "Skip item X, it's not applicable" → Remove from checklist, note why

Iterate until the developer gets to "READY FOR PR."

## Common Code Patterns to Check

| AC Category | What to look for in code |
|-------------|-------------------------|
| Keyboard | `onKeyDown`, `onKeyUp`, `tabIndex`, `role="button"` on non-buttons |
| Focus management | `useRef` + `.focus()`, `autoFocus`, focus trap libraries |
| aria-expanded | State variable wired to `aria-expanded={state}` |
| aria-label / labelledby | Attribute present on interactive elements |
| role="dialog" | On modal container, with `aria-modal="true"` |
| Live regions | `aria-live`, `role="status"`, `role="alert"` |
| Semantic lists | `<ul>`, `<ol>`, `<li>` for list content |
| Headings | `<h1>`–`<h6>` in correct hierarchy |
| Form labels | `<label htmlFor>`, `aria-labelledby`, `aria-label` on inputs |
| Error states | `aria-invalid`, `aria-describedby` pointing to error message |

## What This Skill Does NOT Do

- Does NOT run automated tests (axe, Lighthouse, etc.)
- Does NOT test with screen readers (manual testing required)
- Does NOT verify runtime behavior (focus trapping, live announcements)
- Does NOT replace QA accessibility testing

Items that can't be verified from code are flagged as "ℹ️ Needs Manual Testing."

## Example

**Input:** Developer says "review a11y for PROJ-1234" + shares screenshot

**Agent:**
1. Finds a11y subtask under PROJ-1234
2. Reads the 20 ACs from the subtask description
3. Reads the changed `.tsx` files
4. Analyzes screenshot for drift
5. Reports: 14 handled, 4 missing (with fixes), 1 new (from screenshot), 1 manual

Developer fixes the 4 missing items, agent re-reviews → all green → ships to QA.
