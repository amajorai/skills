---
name: reflect
description: Reflects on the current conversation to extract corrections, repeated issues, and validated approaches, then updates CLAUDE.md (Claude agents) or AGENTS.md (Codex agents) with new rules. Run at any point to capture what the agent should learn from this session.
---

# Reflect

You are auditing the current conversation and distilling it into durable rules for the agent config file. Work through each step in order.


## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx --yes skills update reflect -y 2>/dev/null || true
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Step 1: Detect the Current Agent

Determine which agent is running by checking in this order:

1. Check environment variables:
   - `$env:CLAUDE_CODE_*` or `$env:ANTHROPIC_*` present → **Claude agent**
   - `$env:CODEX_*` or `$env:OPENAI_*` present → **Codex agent**

2. Check which config file exists at the project root:
   - `CLAUDE.md` exists → **Claude agent**
   - `AGENTS.md` exists → **Codex agent**
   - Both exist → use whichever was modified more recently
   - Neither exists → **Claude agent** (default, create CLAUDE.md)

Record: **Agent type** and **Target config file path**.


## Step 2: Read the Target Config File

Read the current contents of the target config file so you can append or update without overwriting existing rules.

If the file does not exist, treat its contents as empty.


## Step 3: Audit the Current Conversation

Review the full conversation that just happened. Identify:

### 3a: Corrections (what went wrong)
Things the user had to correct, redirect, or explicitly tell you not to do:
- Direct corrections: "no, don't do X", "stop doing Y", "that's wrong"
- Implicit corrections: user rewrote something you wrote, user undid a change you made
- Wasted effort: you did work the user immediately discarded

### 3b: Validated approaches (what worked)
Non-obvious choices the user accepted or confirmed:
- User said "yes, exactly" or "perfect" after an unusual approach
- User accepted a non-default choice without pushback
- A technique that resolved a problem that had previously failed

### 3c: Repeated issues
Things that came up multiple times in the conversation:
- You asked the same clarifying question more than once
- The user explained the same constraint more than once
- You made the same mistake twice

### 3d: Revealed constraints
Facts the user told you that aren't obvious from the code:
- Project-specific rules, architectural decisions, team preferences
- Tools or patterns that are off-limits or required
- External dependencies or environment constraints


## Step 4: Determine What Is Worth Persisting

Filter the findings. Only persist something if it meets ALL of these criteria:

1. **Not derivable from code**: can't be learned by reading files
2. **Likely to recur**: would affect future sessions, not just this one
3. **Non-obvious**: a new agent wouldn't know this by default
4. **Actionable**: stated as a clear rule, not vague advice

Discard:
- Task-specific details ("in this PR we did X")
- Things already documented in the config file
- Things that are standard practice (no extra value in restating them)


## Step 5: Draft New Rules

For each item that passed Step 4, write a rule in this format:

```
**[Short rule label]:** <imperative statement of the rule>. [One sentence on why: the incident or preference that drove it.]
```

Examples of good rules:
```
**No DB mocks in tests:** Use real database connections in all tests. The team was burned when mock/prod divergence masked a broken migration.

**Bun over npm:** Always use `bun` for installs and scripts. The project uses Bun throughout; mixing causes lockfile conflicts.

**No trailing summaries:** Don't end responses with "In summary, I did X." The user finds these redundant and prefers terse responses.
```


## Step 6: Merge Into the Config File

Update the target config file:

1. If a **new section** is needed (e.g., `# Behavior`, `# Constraints`, `# Preferences`), add it at the end.
2. If the rule **augments an existing section**, insert it there.
3. If a rule **replaces something outdated**, update the existing entry.
4. Never delete existing rules unless they directly contradict a new one.

Keep the config file readable: no duplicate entries, no bloated prose.


## Step 7: Report Back

Tell the user:

1. **Agent detected:** [Claude / Codex] → updating [file path]
2. **Rules added:** list each new rule in one line
3. **Rules updated:** list any existing rules that were changed
4. **Skipped:** anything from the conversation that didn't meet the persistence criteria and why

If nothing worth persisting was found, say so explicitly rather than adding filler.
