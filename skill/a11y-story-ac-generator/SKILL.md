---
name: a11y-story-ac-generator
description: >
  Generates accessibility acceptance criteria from a Jira user story and optional
  Figma design. Creates a structured a11y checklist covering keyboard, screen reader,
  ARIA, and visual requirements — then offers to add it as a subtask in Jira.
  Use before development starts to shift-left accessibility requirements.
  Trigger phrases: generate a11y ACs, accessibility criteria for this ticket,
  what a11y should the dev cover, a11y acceptance criteria, accessibility checklist
  for story, create a11y subtask.
compatibility: >-
  Requires Atlassian MCP server (atlassian_test_connection, atlassian_manifest,
  atlassian_read, atlassian_write). Optional: Figma MCP server (fetch_figma_node)
  for Figma design analysis. Optional: a11y MCP server (a11y_read, a11y_search)
  for WCAG criterion lookups.
metadata:
  tags:
    - a11y
    - accessibility
    - jira
    - acceptance-criteria
    - shift-left
    - planning
    - wcag
  version: "1.0.0"
  author: akshatagupta1801
  source: community
  lifecycle:
    - planning
  mcp-prerequisites:
    - name: atlassian
      tools:
        - atlassian_test_connection
        - atlassian_manifest
        - atlassian_read
        - atlassian_write
    - name: figma
      optional: true
      tools:
        - fetch_figma_node
    - name: a11y
      optional: true
      tools:
        - a11y_read
        - a11y_search
---

# A11y Story AC Generator

Generate accessibility acceptance criteria from a Jira user story and optional
Figma design link. Produces a grouped checklist covering keyboard interaction,
screen reader behavior, ARIA semantics, and visual requirements — then
automatically creates a Jira subtask with the generated ACs.

This is the **planning-phase** skill in the shift-left accessibility strategy.
It ensures developers know exactly what to build for accessibility before they
write a single line of code.

## Prerequisites

Before proceeding, verify the required MCP servers are connected:

1. **Required:** Atlassian MCP server
   - **Verify:** Run `atlassian_test_connection()` — if Jira shows configured, proceed.
   - **If not connected:** Ask the user to configure the Atlassian MCP server in their MCP configuration.

2. **Optional:** Figma MCP server (for Figma design analysis)
   - Enables: Figma design parsing to identify UI patterns automatically.
   - Without it: Skill works from Jira story ACs alone (less precise but still useful).

3. **Optional:** a11y MCP server (for WCAG criterion lookups)
   - **Verify:** Run `a11y_manifest()` — if it returns operations, a11y MCP is available.
   - Enables: Precise WCAG SC references and technique recommendations.
   - Without it: Skill uses built-in pattern knowledge (still accurate, less detailed).

Do not proceed until the Atlassian MCP server is confirmed accessible.

## When to Use This Skill

Use this skill when:
- A user story is ready for development and needs accessibility requirements
- The user says "generate a11y ACs", "create accessibility checklist", or "what accessibility do I need for this story?"
- During sprint planning or grooming to add a11y requirements to stories
- A developer picks up a story and wants to know the a11y expectations before coding

Do NOT use this skill when:
- The user wants to review existing code for accessibility → use `a11y-review-jira-ac` instead
- The user wants a full WCAG audit of a component → use a WCAG audit skill instead
- The user just wants WCAG information → call `a11y_search` or `a11y_read` directly

## Workflow

### Step 1: Get the Jira Story

Ask the user for the Jira ticket ID (e.g., `PROJ-1234`).

Fetch the story details:

```
atlassian_read(operationId: "j_getIssue", params: {
  issueIdOrKey: "<TICKET_ID>",
  fields: "summary,description,issuetype,status,subtasks,attachment"
})
```

Extract from the response:
- **Summary** — what the feature is
- **Description / Acceptance Criteria** — what behavior is expected
- **Existing subtasks** — check if an a11y subtask already exists (avoid duplicates)

If an accessibility subtask already exists, inform the user and ask whether to
regenerate or skip.

### Step 2: Identify UI Patterns

From the story description and ACs, identify which UI patterns are involved.
Look for keywords and explicit mentions:

| Pattern | Keywords / Signals |
|---------|-------------------|
| Modal / Dialog | "popup", "modal", "dialog", "overlay", "flyout" |
| Form | "input", "field", "submit", "form", "validation", "select", "dropdown" |
| Navigation | "menu", "nav", "tabs", "breadcrumb", "sidebar" |
| Data Table | "table", "grid", "list", "sort", "filter", "pagination" |
| Accordion / Disclosure | "expand", "collapse", "accordion", "toggle", "show/hide" |
| Carousel / Slider | "carousel", "slider", "rotate", "next/previous" |
| Toast / Notification | "toast", "alert", "notification", "snackbar", "banner" |
| Dynamic Content / Live Updates | content appearing/disappearing conditionally, counts updating, async results, view transitions without page reload |
| Tooltip / Popover | "tooltip", "popover", "hover info" |
| Stepper / Wizard | "step", "wizard", "multi-step", "progress" |
| Drag and Drop | "drag", "reorder", "sortable", "move" |
| Autocomplete / Combobox | "autocomplete", "combobox", "typeahead", "search suggestions" |
| Tree View | "tree", "hierarchy", "nested list", "expand nodes" |
| Date Picker | "date", "calendar", "picker" |
| File Upload | "upload", "attach", "file" |
| Rich Text Editor | "editor", "rich text", "formatting" |
| Multi-Section Content | "sections", "steps", "groups", multiple distinct content areas, headings implied |

### Step 3: Analyze Figma Design (if available)

**Option A: Screenshot provided (preferred)**

If the user drags a Figma screenshot or design image into the chat, perform
visual analysis to identify:

- Component types (checkboxes vs radio vs toggles, buttons vs links)
- Whether expand/collapse uses a separate chevron or the whole row is clickable
- Whether it's a modal overlay or inline page content
- Visual hierarchy (what's a heading vs a label vs descriptive text)
- Groupings and sections (how items are visually clustered)
- Interactive elements (what's clickable, what's read-only)
- State indicators (checked, expanded, disabled, selected)
- Potential contrast issues (light grey text, low-contrast borders)
- Touch target sizes (small checkboxes, tiny close buttons)
- Color-only information (red = error without icon/text)

This resolves most ambiguities without needing to ask the developer questions.

**Option B: Figma URL + Figma MCP available**

If the user provides a Figma URL and the Figma MCP server is connected:

```
fetch_figma_node({ url: "<FIGMA_URL>", mode: "skeleton" })
```

From the Figma design tree, identify component types, interactive elements,
visual hierarchy, state variations, and content patterns.

**Option C: No visual reference**

Proceed with story ACs alone. The skill will ask clarifying questions in Step 4
to resolve ambiguities before generating ACs.

Merge all findings (story ACs + visual analysis) before proceeding to Step 4.

### Step 4: Generate Accessibility Acceptance Criteria

**First: Clarify ambiguous patterns.** Do NOT guess — wrong ACs are worse than
no ACs because developers will implement the wrong ARIA pattern.

**Priority order for resolving ambiguity:**

1. Screenshot → analyze visually (resolves most ambiguities without questions)
2. Figma tree → fetch via `fetch_figma_node`
3. Ask clarifying questions (fallback, one batch only):

| Ambiguity | Ask |
|-----------|-----|
| Checkbox + expand/collapse on same row | "Is the checkbox itself the expand trigger, or is there a separate expand button?" |
| "List" could be flat or nested | "Are these items a flat list, or do they have sub-items (tree structure)?" |
| "Select" could be single or multi | "Is this single-select (radio) or multi-select (checkboxes)?" |
| "Popup" could be modal or non-modal | "Does this overlay block interaction with the page behind it (modal)?" |
| Heading vs label ambiguity | "Are these section titles (headings) or labels for interactive elements?" |
| "Disabled" button behavior | "Should the button be hidden, visually disabled but focusable, or removed from tab order?" |
| Auto-triggered content | "Does this content appear automatically, or only after user action?" |

**Always check for dynamic content:** Does any content change without a full
page reload? Conditional display, selection-driven updates, async operations,
SPA view transitions → include Dynamic Content / Live Updates ACs.

**If the a11y MCP is available**, enhance ACs with precise WCAG references:

```
a11y_search(operation: "search_success_criteria", params: { query: "<pattern>" })
a11y_read(operation: "get_techniques_for_sc", params: { sc: "<sc_id>" })
```

#### Output Format

```markdown
## Accessibility Acceptance Criteria

### Keyboard & Focus
- [ ] [Specific keyboard interaction requirement]
- [ ] [Focus management requirement]

### Screen Reader
- [ ] [Accessible name requirement]
- [ ] [State announcement requirement]
- [ ] [Live region / dynamic content requirement]

### ARIA & Semantics
- [ ] [Semantic markup requirement]
- [ ] [ARIA role/attribute requirement]

### Visual & Responsive
- [ ] [Color contrast requirement]
- [ ] [Text resize requirement]
```

### Step 5: Pattern-Specific AC Templates

Use these templates based on identified patterns. Include only what's relevant.

#### Modal / Dialog
```
Keyboard & Focus:
- [ ] Focus moves into the modal when it opens
- [ ] Focus is trapped inside the modal while open (Tab cycles within)
- [ ] Escape key closes the modal
- [ ] Focus returns to the trigger element on close

Screen Reader:
- [ ] Modal has accessible name (aria-labelledby pointing to heading, or aria-label)
- [ ] Opening the modal announces its name and role
- [ ] Content behind the modal is hidden from assistive tech (aria-hidden or inert)

ARIA & Semantics:
- [ ] Container has role="dialog" (or role="alertdialog" if confirmation)
- [ ] aria-modal="true" is set on the dialog container
```

#### Form / Input
```
Keyboard & Focus:
- [ ] All form controls are reachable via Tab
- [ ] Submit can be triggered with Enter key
- [ ] Focus moves to first error field on validation failure

Screen Reader:
- [ ] Every input has a visible, associated label (<label for="..."> or aria-labelledby)
- [ ] Required fields are announced (aria-required="true")
- [ ] Error messages are associated with their field (aria-describedby)
- [ ] Error state is conveyed (aria-invalid="true" when validation fails)

ARIA & Semantics:
- [ ] Form uses <fieldset> + <legend> for related groups (e.g., radio buttons)
- [ ] Help text is linked via aria-describedby
- [ ] Autocomplete attributes are set for common fields (name, email, address)

Visual & Responsive:
- [ ] Error indication does not rely on color alone (icon + text + border)
- [ ] Labels remain visible (no placeholder-only labels)
```

#### Data Table
```
Keyboard & Focus:
- [ ] Interactive cells/rows are focusable and operable via keyboard
- [ ] Sort controls are keyboard accessible
- [ ] Pagination controls are keyboard accessible

Screen Reader:
- [ ] Table has an accessible name (caption or aria-label)
- [ ] Column headers use <th scope="col">
- [ ] Row headers use <th scope="row"> where applicable
- [ ] Sort state is announced (aria-sort="ascending|descending|none")
- [ ] Pagination announces current page context

ARIA & Semantics:
- [ ] Uses semantic <table> markup OR role="grid" with proper ARIA
- [ ] Empty state has meaningful message
```

#### Tabs
```
Keyboard & Focus:
- [ ] Arrow keys move between tabs (Left/Right for horizontal, Up/Down for vertical)
- [ ] Tab key moves focus from tab list to tab panel content
- [ ] Home/End move to first/last tab

Screen Reader:
- [ ] Tab list has accessible name (aria-label on tablist)
- [ ] Selected tab is announced (aria-selected="true")
- [ ] Tab panel is associated with its tab (aria-labelledby)

ARIA & Semantics:
- [ ] Container has role="tablist"
- [ ] Each tab has role="tab"
- [ ] Each panel has role="tabpanel"
- [ ] aria-controls links tab to its panel
```

#### Toast / Notification
```
Screen Reader:
- [ ] Non-critical notifications use role="status" (polite announcement)
- [ ] Critical alerts use role="alert" (assertive announcement)
- [ ] Notification content is announced when it appears

Keyboard & Focus:
- [ ] Dismissible notifications have a keyboard-accessible close button
- [ ] Focus is NOT stolen from the user's current task
- [ ] If action is required, focus moves to the notification

Visual & Responsive:
- [ ] Auto-dismiss timing is sufficient (minimum 5 seconds, or no auto-dismiss for actions)
- [ ] Information is not conveyed by color alone
```

#### Dynamic Content / Live Updates
```
Screen Reader:
- [ ] Content that appears/disappears dynamically is wrapped in an aria-live region
- [ ] Use aria-live="polite" for non-urgent updates (selection counts, step changes)
- [ ] Use aria-live="assertive" only for critical/time-sensitive changes (errors)
- [ ] Status messages use role="status" (SC 4.1.3)
- [ ] Dynamically loaded content announces its arrival without stealing focus
- [ ] Selection count changes are announced (e.g., "3 of 5 selected")

Keyboard & Focus:
- [ ] When new content appears, focus moves to it only if user-initiated
- [ ] If content disappears, focus moves to a logical next element (not lost to <body>)
```

#### Accordion / Disclosure
```
Keyboard & Focus:
- [ ] Enter/Space toggles the accordion panel open/closed
- [ ] Focus remains on the trigger after toggle

Screen Reader:
- [ ] Trigger announces expanded/collapsed state (aria-expanded="true|false")
- [ ] Panel content is associated with trigger (aria-controls)

ARIA & Semantics:
- [ ] Trigger is a <button> (or has role="button")
- [ ] Heading level is appropriate for the page hierarchy
```

#### Multi-Section Content / Headings
```
Content Structure & Headings:
- [ ] View has a clear top-level heading identifying the context
- [ ] Each distinct section has a heading at the correct level (h3 under h2, etc.)
- [ ] Heading hierarchy is sequential — no skipped levels
- [ ] Sections are wrapped in labelled regions (<section aria-labelledby="...">)

Screen Reader:
- [ ] Headings are navigable via screen reader heading shortcuts (H key in JAWS/NVDA)
- [ ] Section landmarks allow "jump to section" navigation
```

#### Autocomplete / Combobox
```
Keyboard & Focus:
- [ ] Arrow keys navigate suggestions list
- [ ] Enter selects the highlighted suggestion
- [ ] Escape closes the suggestions and restores input value
- [ ] Typing filters suggestions without losing focus

Screen Reader:
- [ ] Input has role="combobox" with aria-expanded
- [ ] Suggestions list has role="listbox"
- [ ] Active suggestion is announced (aria-activedescendant)
- [ ] Result count is announced when list updates

ARIA & Semantics:
- [ ] aria-autocomplete="list" or "both" is set
- [ ] aria-controls links input to listbox
```

### Step 6: Self-Validate Before Presenting

Before showing ACs to the user, check:

1. Every interactive element has keyboard ACs
2. Every dynamic change has an announcement AC
3. No element is both a heading AND a label
4. Focus management is complete (modal opens → in; closes → returns; disappears → not lost)
5. WCAG level matches — Level A items are blocking, Level AA important
6. ACs are specific, not generic ("Button is accessible" is useless; "Update objective button has accessible name matching visible text" is actionable)
7. No contradictions

Fix any failures before presenting.

### Step 7: Present and Offer Subtask Creation

Present the ACs, then ask:

> "Would you like me to create an accessibility subtask in Jira under `<TICKET_ID>` with these acceptance criteria?"

If confirmed:

```
atlassian_write(operationId: "j_createIssue", params: {
  body_fields: {
    "project": { "key": "<PROJECT_KEY>" },
    "parent": { "key": "<PARENT_TICKET_ID>" },
    "issuetype": { "name": "Sub-task" },
    "summary": "Accessibility: <short description from parent story>",
    "description": "<formatted ACs as Jira markup>"
  }
})
```

**Jira markup conversion:** `## Heading` → `h3. Heading`, `- [ ] Item` → `* ( ) Item`

After creation: "Created accessibility subtask `<SUBTASK_KEY>` under `<PARENT_KEY>`. The a11y review skill will use this subtask as its reference when reviewing your code later."

### Step 8: Handle Edge Cases

- **No clear UI patterns:** Inform user — a11y ACs are typically needed for user-facing features only
- **Story already has an a11y subtask:** Ask whether to regenerate and update or keep existing
- **No Figma link:** Proceed from story ACs alone; note results may be less precise
- **Multiple UI patterns:** Generate ACs for all, grouped clearly; developer removes inapplicable items

## What This Skill Does NOT Do

- Does NOT review existing code → use `a11y-review-jira-ac`
- Does NOT run automated tests → use axe, Lighthouse, or manual testing
- Does NOT replace accessibility expert review for complex interactions
- Does NOT guarantee WCAG compliance — it provides a starting checklist

## Example

**Input:** `PROJ-1234` — "Quick Add Objective" flyout

**Output:**

```markdown
## Accessibility Acceptance Criteria — PROJ-1234

### Keyboard & Focus
- [ ] Focus moves into the flyout when it opens
- [ ] Tab cycles through all interactive elements inside the flyout
- [ ] Escape key closes the flyout
- [ ] Focus returns to the "Quick Add" trigger button on close
- [ ] All form fields and buttons are reachable via Tab

### Screen Reader
- [ ] Flyout has accessible name via aria-labelledby (pointing to its heading)
- [ ] Opening the flyout announces its name and role
- [ ] Required fields announce their required state
- [ ] Checkbox selections are announced on change
- [ ] Form validation errors are announced and linked to fields

### ARIA & Semantics
- [ ] Flyout container has role="dialog" and aria-modal="true"
- [ ] Objective list uses semantic list markup (<ul>/<li>)
- [ ] Stepper announces current step ("Step 1 of 2") via aria-current or aria-label
- [ ] Content behind the flyout is inert (aria-hidden or inert attribute)

### Visual & Responsive
- [ ] Focus indicator is visible on all interactive elements (min 2px, 3:1 contrast)
- [ ] Error states use icon + text (not color alone)
- [ ] Text remains readable at 200% zoom
```

Then: "Would you like me to create this as a subtask under PROJ-1234?"
