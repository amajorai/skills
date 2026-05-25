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
npx --yes skills update agent-quality -y 2>/dev/null || true
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

Scans React code for a health score (0–100) covering performance, architecture, security, and a11y — flags issues AI agents commonly introduce.

See [references/react-doctor.md](references/react-doctor.md) for install, config, CI setup, and verification.


## Phase 3: react-scan

*Skip if not selected.*

Highlights components causing unnecessary re-renders in the browser with a visual overlay — no code changes needed to start.

See [references/react-scan.md](references/react-scan.md) for install, config, CI setup, and verification.


## Phase 4: react-grab

*Skip if not selected.*

Press ⌘C / Ctrl+C over any browser element to copy its component name, file path, and HTML — paste directly into Claude Code for precise context.

See [references/react-grab.md](references/react-grab.md) for install, config, CI setup, and verification.


## Phase 5: expect

*Skip if not selected.*

Reads the current `git diff`, generates a test plan, and runs it in a real Playwright browser — no selectors or assertions to write.

See [references/expect.md](references/expect.md) for install, config, CI setup, and verification.


## Phase 6: agentation

*Skip if not selected.*

Desktop app that annotates UI elements with CSS selectors, component paths, and file locations — exports structured markdown for the agent.

See [references/agentation.md](references/agentation.md) for install, config, CI setup, and verification.


## Phase 7: dev3000

*Skip if not selected.*

AI-powered debugging and development monitoring from Vercel Labs. Designed for Next.js and Vercel-hosted projects.

Visit [dev3000.ai](https://dev3000.ai/) for current installation instructions: the setup flow is project-specific. Recommended for Next.js projects deployed on Vercel.


## Phase 8: Update CLAUDE.md

Add a Quality Tools section to CLAUDE.md for each installed tool — see [references/claude-md-template.md](references/claude-md-template.md).


## Completion Report

For each installed tool:
- Tool name and version installed
- How to invoke it (command or workflow step)
- CI integration added (yes/no)
- CLAUDE.md updated (yes/no)
- Verification: passed/failed
