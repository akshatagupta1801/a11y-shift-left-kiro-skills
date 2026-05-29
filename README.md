# A11y Shift-Left AI Agent Skills

Two AI agent skills that bring accessibility requirements into your development workflow **before** code is written and **before** PRs are raised.

These skills work with any AI coding assistant that supports MCP (Model Context Protocol) servers — including Kiro, Cursor, Copilot, Windsurf, or any MCP-compatible agent.

## Skills

### 1. a11y-story-ac-generator

**Phase:** Planning (before development)

Generates accessibility acceptance criteria from a Jira user story and optional Figma design. Produces a grouped checklist covering keyboard interaction, screen reader behavior, ARIA semantics, and visual requirements — then automatically creates a Jira subtask with the generated ACs.

**Trigger phrases:**
- "generate a11y ACs"
- "accessibility criteria for this ticket"
- "what a11y should the dev cover"
- "create a11y subtask"

### 2. a11y-review-jira-ac

**Phase:** Development (before PR)

Reviews code changes against the accessibility acceptance criteria in a Jira a11y subtask. Validates each AC against the actual code, flags missing implementations with copy-paste fixes, and catches new elements added since the ACs were written.

**Trigger phrases:**
- "review my a11y changes"
- "check accessibility"
- "validate a11y"
- "did I cover all a11y requirements"

## How They Work Together

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────┐
│  User Story in  │────▶│  a11y-story-ac   │────▶│ A11y Subtask│
│     Jira        │     │    -generator     │     │  in Jira    │
└─────────────────┘     └──────────────────┘     └──────┬──────┘
                                                         │
                                                         ▼
┌─────────────────┐     ┌──────────────────┐     ┌─────────────┐
│  Ready for PR   │◀────│  a11y-review     │◀────│  Developer  │
│  (all green)    │     │    -jira-ac       │     │  codes it   │
└─────────────────┘     └──────────────────┘     └─────────────┘
```

## Prerequisites

### Required
- **Atlassian MCP server** — for reading/writing Jira issues
  - Tools needed: `atlassian_test_connection`, `atlassian_manifest`, `atlassian_read`, `atlassian_write`

### Optional
- **Figma MCP server** — for analyzing Figma designs (improves AC generation accuracy)
  - Tools needed: `fetch_figma_node`
- **A11y MCP server** — for WCAG criterion lookups and technique references
  - Tools needed: `a11y_read`, `a11y_search`

## Installation

These are AI agent prompt files. How you install them depends on your AI coding assistant:

### Generic (any MCP-compatible agent)
Copy the skill markdown files and include them as system prompts or context when interacting with your AI assistant.

### Kiro
Copy into your workspace:
```
.kiro/skills/a11y-story-ac-generator.md
.kiro/skills/a11y-review-jira-ac.md
```

### Cursor / Windsurf
Add as rules or context files per your tool's documentation.

### Custom Agent
Include the SKILL.md content as part of your agent's system prompt or tool instructions.

## MCP Server Configuration

You'll need an Atlassian MCP server configured. Example configuration:

```json
{
  "mcpServers": {
    "atlassian": {
      "type": "http",
      "url": "YOUR_ATLASSIAN_MCP_URL",
      "headers": {
        "X-Jira-Token": "YOUR_JIRA_PAT"
      }
    }
  }
}
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

[MIT](LICENSE)

## Author

Akshata Gupta ([@akshatagupta1801](https://github.com/akshatagupta1801))
