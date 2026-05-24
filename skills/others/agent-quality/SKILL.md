---
name: agent-quality
description: Set up quality tools designed specifically for AI-generated code: react-doctor (health scoring), react-scan (re-render detection), react-grab (element context capture), expect (browser testing from git diffs), and agentation (UI annotation for agents). Interviews the user to install only what's relevant.
argument-hint: <leave blank to start the interview>
---

# Agent Quality

You are setting up quality tools that catch the specific problems AI coding agents introduce. Start with an interview to install only what's relevant.


## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx skills update agent-quality -y
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Phase 1: Interview

Present the tools and ask the user which to set up. Show this summary first:


**Available tools:**

| Tool | What it does | Best for |
|------|-------------|----------|
| **react-doctor** | Scans React code and gives a health score (0–100) with diagnostics across performance, architecture, security, and a11y | React / Next.js / React Native projects |
| **react-scan** | Highlights components causing unnecessary re-renders in real time: no code changes needed | Any React app with performance issues |
| **react-grab** | Press ⌘C / Ctrl+C over any UI element to copy its component name, file path, and HTML to clipboard | Giving an agent precise UI element context |
| **expect** | Reads your git diff, generates a test plan, and runs it in a real browser with Playwright: no test scripts to maintain | Any web app with a UI |
| **agentation** | Desktop app that annotates UI elements with CSS selectors, component paths, and file locations for agents | Detailed UI feedback workflows |
| **dev3000** | AI-powered debugging and dev monitoring from Vercel Labs | Vercel / Next.js projects |


Ask in one batch:

- **Which tools** do you want to set up? (can select multiple)
- **React-doctor / react-scan / react-grab**: Is this a React, Next.js, or React Native project?
- **Expect**: What URL does the app run on locally (e.g. `http://localhost:3000`)? Should it run in CI too?
- **Agentation**: Do you want MCP integration so the agent can respond to annotations in real time?

Only proceed with the tools the user selects.


## Phase 2: react-doctor

*Skip if not selected.*

### What it catches
Issues that AI agents commonly introduce: unnecessary re-renders, missing keys, overuse of `useEffect`, inline object/function props breaking memoization, missing error boundaries, security issues, and a11y gaps.

### Install & run

```bash
# Run once (no install needed):
npx react-doctor@latest

# Install for agent use (adds to agent context):
npx react-doctor@latest install
```

The `install` command configures Claude Code / Cursor / Codex to automatically run react-doctor checks. After installing, the agent will reference health scores when making changes.

### Scoring

| Score | Status |
|-------|--------|
| 75–100 | Great |
| 50–74 | Needs work |
| < 50 | Critical |

### CI integration (optional)

Add to `.github/workflows/ci.yml`:

```yaml
- name: React Doctor
  run: npx react-doctor@latest --diff origin/main --json > react-doctor-report.json
```

Use `--diff origin/main` to scan only changed files on a PR. Add `--json` for machine-readable output.

### Config (optional)

Create `react-doctor.config.json` to suppress rules that don't apply:

```json
{
  "ignore": ["no-inline-styles"],
  "threshold": 70
}
```

### Verify

Run `npx react-doctor@latest` and confirm:
- [ ] A score is printed (0–100)
- [ ] Diagnostics list specific files and line numbers
- [ ] `--diff origin/main` returns only changed-file issues


## Phase 3: react-scan

*Skip if not selected.*

### What it catches

Unnecessary re-renders caused by inline functions, inline objects, and missing memoization. Highlights offending components directly in the browser with a visual overlay: no code changes needed to start seeing problems.

### Install & run

```bash
npx -y react-scan@latest init
```

The init command auto-detects your framework (Next.js, Vite, Remix, etc.) and adds the necessary script or import.

**Manual install:**
```bash
bun add -d react-scan
```

Then add to your root layout (Next.js example):
```tsx
import { scan } from 'react-scan'
if (typeof window !== 'undefined') scan({ enabled: true })
```

### Use

Open the app in the browser: a floating toolbar appears in the corner. Components that re-render unnecessarily are highlighted in real time. Click a highlighted component to see why it re-rendered.

**Common fixes react-scan surfaces:**
- Inline functions as props: `onClick={() => ...}` → `useCallback`
- Inline objects as props: `style={{ color: 'red' }}` → extract to a constant or `useMemo`
- Missing `React.memo` on expensive pure components

### Add to CLAUDE.md

```markdown
## Performance
react-scan is installed. Run the dev server and open the app in the browser to see re-render highlights.
Before optimizing a component, confirm it's actually highlighted by react-scan first.
```

### Verify

- [ ] Dev server starts without errors after init
- [ ] Toolbar appears in the browser
- [ ] Triggering a state change highlights the re-rendering components
- [ ] Fixing an inline prop removes the highlight


## Phase 4: react-grab

*Skip if not selected.*

### What it does

Hover over any element in the browser and press **⌘C** (Mac) or **Ctrl+C** (Windows/Linux) to copy the element's full context to clipboard:
- React component name
- Source file path and line number
- HTML source

Paste directly into Claude Code instead of describing the element: gives the agent the exact location without guessing.

### Install

```bash
npx grab@latest init -y
```

The CLI detects your framework and configures the initialization automatically. Supported: Next.js (app and pages router), Vite, Webpack.

**Customize (optional):**
```bash
npx grab@latest config
```

Options: activation key, toggle vs. hold mode, context line count.

### Use

1. Start the dev server
2. Open the app in the browser
3. Hover over any UI element
4. Press ⌘C / Ctrl+C
5. Paste into Claude Code: the agent receives component name, file path, and HTML

Instead of: *"Fix the button in the sidebar"*
The agent receives: *"Component: `<SidebarButton>` at `src/components/sidebar/SidebarButton.tsx:42` - `<button class="sidebar-btn primary">Submit</button>`"*

### Add to CLAUDE.md

```markdown
## Element Context
react-grab is installed. Hover over any UI element and press ⌘C / Ctrl+C to copy its component name, file path, and HTML.
Paste the output here when referencing a specific UI element.
```

### Verify

- [ ] Dev server starts without errors after init
- [ ] Hovering over a component and pressing ⌘C / Ctrl+C copies context to clipboard
- [ ] Pasted output includes component name, file path, and HTML
- [ ] Pasting into Claude Code results in the agent finding the right file without guessing


## Phase 5: expect

*Skip if not selected.*

### What it does

Reads your current `git diff`, generates a test plan covering the changes, and executes it in a real Playwright browser. No selectors or assertions to write: the agent figures it out from the diff.

Catches: broken hover states, missing links, dead buttons, performance regressions (LCP, animation frames), CSRF issues, missing metadata.

### Install

```bash
npm install -g expect-cli
```

### Run

```bash
# Basic run: reads git diff, asks for confirmation, runs in browser:
/expect

# Skip confirmation:
/expect -y

# Custom instructions:
/expect -m "focus on the checkout flow"

# CI mode (no interactive prompts):
/expect --ci
```

### CI integration (optional)

```yaml
- name: Expect browser tests
  run: expect-cli --ci --agent claude
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Add to CLAUDE.md

```markdown
## Testing
After any UI change, run `/expect` to verify the change in a real browser.
Use `/expect -m "<focus area>"` to target a specific flow.
```

### Verify

Make a small visible change (e.g., change a button label), then run `/expect -y` and confirm:
- [ ] expect reads the git diff
- [ ] Generates a test plan referencing the changed component
- [ ] Opens a browser and runs through the plan
- [ ] Reports pass/fail per test step


## Phase 6: agentation

*Skip if not selected.*

### What it does

A desktop app you run alongside the browser. Hover over any UI element to see its component name highlighted. Click to annotate it with feedback. Export structured markdown that gives the agent:

- CSS selectors (`.sidebar > button.primary`)
- React component hierarchy
- Source file paths
- Computed styles
- Your feedback with intent and priority

This replaces vague instructions like "fix the blue button" with precise, actionable context.

### Install

Download from [agentation.com](https://www.agentation.com/): free for individual and internal team use.

### Basic workflow

1. Open Agentation alongside your browser
2. Activate annotation mode
3. Hover over elements to see component names
4. Click to annotate with feedback and priority
5. Copy the generated markdown and paste into Claude Code

### MCP integration (optional: real-time sync)

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

### Add to CLAUDE.md

```markdown
## UI Feedback
Use Agentation to annotate UI issues. Export the markdown and paste it here.
Annotations include CSS selectors and component paths: use them to find the exact code to change.
```

### Verify

- [ ] Agentation opens and activates correctly
- [ ] Hovering over elements highlights component names
- [ ] Annotating an element exports correct CSS selector and file path
- [ ] Pasting the export into Claude Code results in precise file edits (not guessed locations)


## Phase 7: dev3000

*Skip if not selected.*

### What it does

AI-powered debugging and development monitoring from Vercel Labs. Designed for Next.js and Vercel-hosted projects.

### Install

Visit [dev3000.ai](https://dev3000.ai/) for current installation instructions: the setup flow is project-specific.

### Recommended for

- Next.js projects deployed on Vercel
- Teams that want AI-assisted debugging integrated into their development workflow


## Phase 8: Update CLAUDE.md

Add a **Quality Tools** section to the project `CLAUDE.md` for each installed tool:

```markdown
## Quality Tools

### react-doctor (if installed)
Run `npx react-doctor@latest` after making React changes to check the health score.
Target: score ≥ 75. Use `--diff origin/main` on PRs to scan only changed files.

### react-scan (if installed)
Open the app in the browser: the toolbar shows re-render highlights in real time.
Only optimize components that react-scan confirms are actually re-rendering unnecessarily.

### react-grab (if installed)
Hover over any UI element and press ⌘C / Ctrl+C to copy its component name, file path, and HTML.
Paste the output here when referencing a specific UI element: don't describe it vaguely.

### expect (if installed)
Run `/expect` after any UI change to verify in a real browser.
Use `/expect -m "<area>"` to focus on a specific flow.

### agentation (if installed)
For UI feedback, use Agentation to annotate elements and paste the export here.
Annotations include CSS selectors and file paths: use them directly.
```


## Completion Report

For each installed tool:
- Tool name and version installed
- How to invoke it (command or workflow step)
- CI integration added (yes/no)
- CLAUDE.md updated (yes/no)
- Verification: passed/failed
