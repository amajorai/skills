---
name: context
description: Set up the best context tools for AI coding agents: Context7 (live library docs via MCP), opensrc (real package source code), and a project CLAUDE.md. Use when starting a new project or when the agent keeps hallucinating outdated API usage.
argument-hint: <project path or leave blank for current directory>
---

# Context

You are setting up the context infrastructure that makes an AI coding agent accurate and well-informed. Work through each phase in order.

**Project:** {{args}}


## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx --yes skills update amajorai/skills -y 2>/dev/null || true
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Phase 1: Interview

Ask the user (combine related questions):

- **Primary languages/frameworks**: What stack is this project? (Needed to know which library docs matter most)
- **Context7**: Install as a project-level MCP server (`.claude/settings.json`) or user-level (`~/.claude/settings.json`)?
- **opensrc**: Is Node.js 18+ installed? (required for opensrc CLI)
- **CLAUDE.md**: Does a `CLAUDE.md` already exist for this project? Should we create or update one?


## Phase 2: Context7: Live Library Docs

Context7 is an MCP server that pulls up-to-date, version-specific library documentation directly into the agent's context. It eliminates hallucinated APIs and outdated usage patterns.

### Install

**Option A: Automatic (recommended):**
```bash
# Run the Context7 installer and select Claude Code:
npx @upstash/context7-mcp@latest init --claude

# Or register it directly with the Claude Code CLI (user-level):
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
```

**Option B: Manual config:**

Add to `.claude/settings.json` (project-level) or `~/.claude/settings.json` (user-level):

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

Restart Claude Code after adding the config.

### Verify

Ask Claude: `What is the latest way to do server actions in Next.js? use context7`

If Context7 is working, Claude will fetch live docs from the official Next.js documentation and cite the version. If it falls back to training data, the MCP config is not loaded yet.

### How to use

Add `use context7` to any prompt involving a library:

```
How do I set up Drizzle ORM with Postgres? use context7
Create a Hono middleware that validates a JWT. use context7
What changed in React 19 for forms? use context7
```

To target a specific library: `use context7 library /vercel/next.js`


## Phase 3: opensrc: Real Package Source Code

opensrc gives the agent access to the actual source code of any npm, PyPI, or Rust crate: not just types or docs. Useful when you need to understand exactly how a library works internally, trace a bug into a dependency, or find real usage examples.

### Install

```bash
npm install -g opensrc
```

Requires Node.js 18+ and Git. Packages are fetched on first use and cached locally.

### Verify

```bash
opensrc path zod
# Should return a local cache path like: /home/user/.opensrc/npm/zod@3.x.x
```

### How to use

```bash
# Read a specific file from a package
cat $(opensrc path drizzle-orm)/src/pg-core/columns/common.ts

# Search a package's source
rg "createServer" $(opensrc path hono)/src

# Explore a package's structure
ls $(opensrc path better-auth)/packages/better-auth/src
```

Tell the agent: "Use `opensrc path <package>` to read the source of <package> before implementing."

Add to `CLAUDE.md` so the agent uses it automatically: see Phase 4.


## Phase 4: CLAUDE.md: Project Context File

A `CLAUDE.md` at the project root is loaded into Claude Code's context on every session. It is the most reliable way to give the agent persistent, project-specific knowledge.

### Create or update `CLAUDE.md` with:

```markdown
# Project Context

## Stack
[List the key frameworks, libraries, and tools with their versions]

## Architecture
[Brief description of the main components and how they connect]

## Key Conventions
[Coding conventions, naming patterns, important file locations]

## Context Tools
- **Library docs**: Add `use context7` to any prompt about a library API
- **Package source**: Use `opensrc path <package>` to read a dependency's actual source code
  - Example: `cat $(opensrc path hono)/src/router/trie-router/router.ts`

## Commands
- `bun dev` - start dev server
- `bun test` - run tests
- `bun run db:migrate` - run migrations
[Add the actual commands for this project]
```

If a `CLAUDE.md` already exists, add the Context Tools section to it rather than overwriting.


## Phase 5: Explore the Project

With context tools in place, run a structured codebase exploration so the agent starts with accurate project knowledge. Spawn **3 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | **Stack inventory**: read `package.json` / `Cargo.toml` / `requirements.txt`, identify all key dependencies and their versions |
| 2 | **Architecture**: read entry points, main config files, directory structure, understand how the pieces connect |
| 3 | **Conventions**: find coding patterns, naming conventions, how tests are structured, any existing documentation |

Synthesize findings into a **Context Summary** and offer to write it into `CLAUDE.md`.


## Phase 6: Verify Everything Works

Run these checks:

- [ ] Context7 MCP responds: ask a library question with `use context7` and confirm it cites live docs
- [ ] opensrc resolves a package: run `opensrc path <main framework>` and confirm a path is returned
- [ ] `CLAUDE.md` exists with stack, architecture, conventions, and context tool instructions
- [ ] Restart Claude Code and confirm MCP servers load (check `/mcp` status)


## Completion Report

- Context7: installed (project-level or user-level), verified working
- opensrc: installed, verified working
- CLAUDE.md: created or updated with project context
- Key libraries where Context7 will be most useful (list)
