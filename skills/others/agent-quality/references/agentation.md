# agentation Reference

## What it does

A desktop app you run alongside the browser. Hover over any UI element to see its component name highlighted. Click to annotate it with feedback. Export structured markdown that gives the agent:

- CSS selectors (`.sidebar > button.primary`)
- React component hierarchy
- Source file paths
- Computed styles
- Your feedback with intent and priority

This replaces vague instructions like "fix the blue button" with precise, actionable context.

## Install

Download from [agentation.com](https://www.agentation.com/): free for individual and internal team use.

## Basic workflow

1. Open Agentation alongside your browser
2. Activate annotation mode
3. Hover over elements to see component names
4. Click to annotate with feedback and priority
5. Copy the generated markdown and paste into Claude Code

## MCP integration (optional: real-time sync)

If the user wants MCP integration, add to `.claude/settings.json`:

```json
{
  "mcpServers": {
    "agentation": {
      "command": "npx",
      "args": ["-y", "agentation-mcp", "server"]
    }
  }
}
```

With MCP enabled, Claude Code receives annotations in real time and can ask clarifying questions about UI feedback: turning it into a two-way conversation rather than one-directional input.

## Add to CLAUDE.md

```markdown
## UI Feedback
Use Agentation to annotate UI issues. Export the markdown and paste it here.
Annotations include CSS selectors and component paths: use them to find the exact code to change.
```

## Verify

- [ ] Agentation opens and activates correctly
- [ ] Hovering over elements highlights component names
- [ ] Annotating an element exports correct CSS selector and file path
- [ ] Pasting the export into Claude Code results in precise file edits (not guessed locations)
