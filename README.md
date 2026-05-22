# A Major Skills

A collection of reusable AI coding agent skills for **Claude Code** and **Codex**. Each skill is a single unified `SKILL.md` file with common phases up top and platform-specific notes inline.

```
skills/
  <skill-name>/
    SKILL.md
.claude-plugin/
  plugin.json
  marketplace.json
install.sh
```

## Skills

| Skill | What it does |
|-------|-------------|
| [`ship`](skills/ship/SKILL.md) | Full-cycle workflow: interview, explore, plan, implement, verify, simplify, security, final verify |

## Installation

### skills.sh (recommended)

```bash
npx skills add amajorai/skills
```

Installs all skills and automatically configures them for whichever coding agents you have installed (Claude Code, Codex, Cursor, and 50+ others).

Install a single skill:

```bash
npx skills add amajorai/skills/skills/ship
```

### Claude Code plugin

```
/plugin marketplace add amajorai/skills
/plugin install amajor-skills@amajorai
```

Invoked as `/amajor-skills:ship <task>`.

### install.sh (one-liner)

```bash
curl -fsSL https://raw.githubusercontent.com/amajorai/skills/main/install.sh | bash
```

```bash
# Codex
curl -fsSL https://raw.githubusercontent.com/amajorai/skills/main/install.sh | bash -s -- --codex
```

Or clone and run manually:

```bash
git clone https://github.com/amajorai/skills.git
cd skills

./install.sh           # Claude Code, copies to ~/.claude/skills/, invoke as /ship
./install.sh --codex   # Codex, copies to ~/.codex/skills/, invoke as $ship
```

### Copy a single skill

```bash
# Claude Code
cp skills/ship/SKILL.md ~/.claude/skills/ship.md

# Codex
mkdir -p ~/.codex/skills/ship && cp skills/ship/SKILL.md ~/.codex/skills/ship/SKILL.md
```
