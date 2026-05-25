# react-grab Reference

## What it does

Hover over any element in the browser and press **⌘C** (Mac) or **Ctrl+C** (Windows/Linux) to copy the element's full context to clipboard:
- React component name
- Source file path and line number
- HTML source

Paste directly into Claude Code instead of describing the element: gives the agent the exact location without guessing.

## Install

```bash
npx -y grab@latest init
```

The CLI detects your framework and configures the initialization automatically. Supported: Next.js (app and pages router), Vite, Webpack.

**Optional MCP integration** (lets the agent respond to grabbed elements):
```bash
npx -y grab@latest add mcp
```

## Use

1. Start the dev server
2. Open the app in the browser
3. Hover over any UI element
4. Press ⌘C / Ctrl+C
5. Paste into Claude Code: the agent receives component name, file path, and HTML

Instead of: *"Fix the button in the sidebar"*
The agent receives: *"Component: `<SidebarButton>` at `src/components/sidebar/SidebarButton.tsx:42` - `<button class="sidebar-btn primary">Submit</button>`"*

## Add to CLAUDE.md

```markdown
## Element Context
react-grab is installed. Hover over any UI element and press ⌘C / Ctrl+C to copy its component name, file path, and HTML.
Paste the output here when referencing a specific UI element.
```

## Verify

- [ ] Dev server starts without errors after init
- [ ] Hovering over a component and pressing ⌘C / Ctrl+C copies context to clipboard
- [ ] Pasted output includes component name, file path, and HTML
- [ ] Pasting into Claude Code results in the agent finding the right file without guessing
