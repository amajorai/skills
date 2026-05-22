---
name: distill-skill
description: Scans conversation history (current session and/or past transcripts) to identify repeated multi-step workflows, then extracts them into new reusable Claude Code skills written to .claude/skills/<name>.md. Turns what you keep doing manually into a one-word command.
---

# distill-skill — Repeated Workflows → New Skills

You are mining conversation history to find workflows that were done more than once and are good candidates to become reusable skills. Work through each step.

---

## Step 1: Scope — What to Scan

Determine the scan scope:

- **Current session only** (default if no args): scan only this conversation
- **All past transcripts** (if args say "history" or "all"): also scan past conversation JSONL files
- **Specific topic** (if args name a topic): filter for that domain only

**Args:** {{args}}

---

## Step 2: Locate Transcript Files (if scanning history)

If scanning past transcripts:

**Claude Code (Windows):**
```powershell
$transcriptDir = "$env:USERPROFILE\.claude\projects"
Get-ChildItem $transcriptDir -Recurse -Filter "*.jsonl" | Sort-Object LastWriteTime -Descending | Select-Object -First 20
```

**Claude Code (macOS/Linux):**
```bash
find ~/.claude/projects -name "*.jsonl" | xargs ls -t | head -20
```

Read the most recent 10–20 files. Skip files under 1KB.

---

## Step 3: Read Existing Skills

List all skills already defined so you don't create duplicates:

```bash
# List existing skill files
ls .claude/skills/
```

Note what each existing skill covers (from its `description:` frontmatter field).

---

## Step 4: Identify Candidate Workflows

Scan the conversation(s) for sequences that:

1. **Appeared more than once** — the same type of task was done in 2+ turns or 2+ sessions
2. **Have clear phases** — the workflow has a beginning, middle, and end
3. **Are general enough** — the workflow applies to any project, not just this specific ticket
4. **Are not already a skill** — not covered by existing `.claude/skills/*.md` files

### Signals to look for:
- The user invoked the same sequence of steps manually in multiple sessions
- The agent did the same multi-step setup procedure more than once
- The user said "do the same thing we did last time for X"
- A complex task took many turns but followed a repeatable pattern
- The user copy-pasted a workflow from an earlier session

### Workflow types worth extracting:
- **Setup flows** — install, configure, authenticate a tool
- **Scaffold flows** — create a new thing (project, module, config file) from scratch
- **Audit flows** — check something for correctness, security, or completeness
- **Deploy flows** — get code running somewhere
- **Integration flows** — connect two systems together

### Do NOT extract:
- One-off tasks tied to a specific bug or feature
- Workflows that are just "run these 2 commands" — too simple to be a skill
- Workflows that already have a `/slash-command` equivalent

---

## Step 5: Score Each Candidate

For each candidate, score it:

| Factor | Score |
|--------|-------|
| Appeared in 3+ sessions | +3 |
| Appeared in 2 sessions | +1 |
| Has 4+ distinct steps | +2 |
| Applicable to any project (not this one) | +2 |
| Would save 5+ minutes if automated | +2 |
| User explicitly said "we always do this" | +3 |

Build skills for candidates scoring **5 or higher**. Skip the rest.

---

## Step 6: Design Each Skill

For each candidate that passes Step 5:

**Name:** Short, kebab-case, verb-noun. Describes what the skill *does*. Examples: `setup-drizzle`, `audit-permissions`, `scaffold-api-route`.

**Does it need `{{args}}`?** Only if the workflow needs a parameter (e.g., project name, target URL). If it's always the same, omit args.

**Skill type:**
- **Setup**: One-time configuration
- **Scaffold**: Creates something new from scratch
- **Workflow**: Multi-phase with decision points
- **Audit**: Checks something for quality or correctness
- **Deploy**: Gets code running somewhere

---

## Step 7: Write Each Skill File

For each skill, write `.claude/skills/<name>.md`:

```markdown
---
name: <kebab-case-name>
description: <One sentence: what it does, when to use it, key tech names for precise triggering.>
argument-hint: <optional: what the user should pass>
---

# <name> — <Human-Readable Title>

<One paragraph: what this skill does, what it produces, and why it exists. Mention that it was distilled from repeated usage patterns.>

---

## Phase 1: <First Phase Name>

<Steps — concrete, executable, with code blocks for every command>

---

## Phase 2: <Second Phase Name>

<Steps>

---

## Completion Checklist

- [ ] <Verifiable outcome 1>
- [ ] <Verifiable outcome 2>
```

### Body writing rules:
- **Every command in a code block** — even single-line ones
- **Decision branches preserved** — if the workflow varies by environment or choice, keep the branches
- **Concrete, not abstract** — no "configure as appropriate", always say what to actually do
- **`{{args}}` only where needed** — use it for the parameter that varies across invocations
- **No filler** — no "Great! Now we'll..." transitions

---

## Step 8: Report Back

Tell the user:

1. **Sessions scanned:** [count] and date range
2. **Candidates found:** [count] total, [count] passed the scoring threshold
3. **Skills created:** list each new skill with file path and one-line description
4. **Skipped candidates:** what you found but didn't extract, and why (score too low, already a skill, too project-specific)

For each new skill, show how to invoke it: `/<name>` or `/<name> <args>`.

If no candidates passed the threshold, tell the user what the highest-scoring candidate was and why it didn't qualify — they may want to lower the bar.
