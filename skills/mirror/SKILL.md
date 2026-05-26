---
name: mirror
description: Scans all past conversation transcripts for the current project and agent type, identifies recurring patterns and improvement opportunities across sessions, then updates the project's agent config file (CLAUDE.md or AGENTS.md). Gives the agent a view of its own history.
---

# Mirror

You are scanning the project's conversation history to find recurring patterns across sessions and distill them into durable rules. This is a deeper version of `/reflect` - it looks back at every past conversation, not just the current one.


## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx --yes skills update amajorai/skills -y 2>/dev/null || true
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Step 1: Detect the Current Agent

Determine which agent is running:

1. Check environment variables:
   - `$env:CLAUDE_CODE_*` or `$env:ANTHROPIC_*` present → **Claude agent**
   - `$env:CODEX_*` or `$env:OPENAI_*` present → **Codex agent**

2. Check which config file exists at the project root:
   - `CLAUDE.md` → Claude agent
   - `AGENTS.md` → Codex agent
   - Both → use the more recently modified one
   - Neither → Claude agent (default)

Record: **Agent type**, **Target config file**, **Transcript directory**.


## Step 2: Locate Transcript Files

Find where past conversation transcripts are stored for this project. Claude Code keeps each project's transcripts in a subdirectory of `~/.claude/projects/` whose name is the project's absolute path with path separators replaced by dashes (e.g. `D:\Code\md\vibemd` → `D--Code-md-vibemd`). Prefer that project-specific subdirectory; only fall back to scanning all of `~/.claude/projects/` if you cannot identify it.

**Claude Code (Windows):**
```powershell
# Transcripts live in the project's path-derived directory under ~/.claude/projects/
$transcriptDir = "$env:USERPROFILE\.claude\projects"
Get-ChildItem $transcriptDir -Recurse -Filter "*.jsonl" | Sort-Object LastWriteTime -Descending
```

**Claude Code (macOS/Linux):**
```bash
# List newest transcripts first (most recently modified at the top)
find ~/.claude/projects -name "*.jsonl" -print0 2>/dev/null \
  | xargs -0 ls -t 2>/dev/null | head -50
```

**Codex:**
```bash
# Codex stores sessions in ~/.codex/sessions/ or similar
find ~/.codex \( -name "*.json" -o -name "*.jsonl" \) -print0 2>/dev/null \
  | xargs -0 ls -t 2>/dev/null | head -50
```

Read the most recent 10–20 transcript files (or all of them if fewer than 10 exist). Skip files that are empty or under 1KB.


## Step 3: Read the Transcripts

For each transcript file, extract the conversation turns. JSONL files have one JSON object per line. Each line typically looks like:

```json
{"type": "user", "message": "...", "timestamp": "..."}
{"type": "assistant", "message": "...", "timestamp": "..."}
```

Focus on:
- User messages that contain corrections, redirections, or explicit instructions
- Assistant messages that preceded those corrections (what triggered them)
- Turns where the user accepted or confirmed a non-obvious approach

You do not need to read transcripts exhaustively: scan for signal, not every word.


## Step 4: Read the Current Config File

Read the existing contents of the target config file so you can see what rules are already documented and avoid adding duplicates.


## Step 5: Find Cross-Session Patterns

Look across all transcripts for signals that appeared in **multiple separate sessions**. A single-session incident is noise; a pattern across sessions is signal.

### 5a: Recurring corrections
The user had to correct the same type of mistake in 2+ sessions:
- Same wrong tool used
- Same bad default assumed
- Same explanation given twice about a constraint

### 5b: Repeated instructions
The user gave the same explicit instruction in 2+ sessions:
- "Always use X" or "Never do Y" appearing in multiple chats
- The same environment fact explained more than once

### 5c: Validated non-obvious choices
An unusual approach appeared and was accepted in 2+ sessions without comment:
- A non-standard pattern the agent kept using and the user kept approving
- A workflow or tool choice that was questioned once and then settled

### 5d: Blind spots
Types of problems where the agent consistently needed more turns than expected:
- Repeatedly misunderstood the same domain concept
- Consistently needed the user to clarify the same type of ambiguity


## Step 6: Score and Filter

Rank each finding by:
- **Frequency**: how many sessions it appeared in (higher = more valuable)
- **Severity**: how much wasted effort did it cause
- **Specificity**: is it actionable as a concrete rule

Discard anything that:
- Only happened once
- Is already documented in the config file
- Is standard practice (not project-specific)
- Cannot be expressed as a clear rule


## Step 7: Draft Rules

For each pattern that passes Step 6, write a rule:

```
**[Label]:** <imperative rule>. [Why: what incident or pattern drives this.]
```

Group related rules under a section header if the config file uses sections.


## Step 8: Merge Into the Config File

Update the target config file:
1. Add new rules to the appropriate section (or create a new section at the end)
2. Strengthen any existing rule that the history confirms is important
3. Remove any rule that the transcript history shows was consistently wrong or ignored
4. Keep the file readable: no duplicate entries


## Step 9: Report Back

Tell the user:

1. **Sessions scanned:** [count] transcripts from [date range]
2. **Patterns found:** [count] cross-session patterns
3. **Rules added:** list each new rule
4. **Rules updated:** list any changes to existing rules
5. **Top insight:** one sentence on the most important thing found

If the history is too sparse (fewer than 3 sessions), say so and suggest running `/reflect` instead.
