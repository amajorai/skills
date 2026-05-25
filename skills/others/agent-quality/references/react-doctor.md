# react-doctor Reference

## What it catches

Issues that AI agents commonly introduce: unnecessary re-renders, missing keys, overuse of `useEffect`, inline object/function props breaking memoization, missing error boundaries, security issues, and a11y gaps.

## Install & run

```bash
# Run once (no install needed):
npx react-doctor@latest

# Install for agent use (adds to agent context):
npx react-doctor@latest install
```

The `install` command configures Claude Code / Cursor / Codex to automatically run react-doctor checks. After installing, the agent will reference health scores when making changes.

## Scoring

| Score | Status |
|-------|--------|
| 75–100 | Great |
| 50–74 | Needs work |
| < 50 | Critical |

## CI integration (optional)

Add to `.github/workflows/ci.yml`:

```yaml
- name: React Doctor
  run: npx react-doctor@latest --diff origin/main --json > react-doctor-report.json
```

Use `--diff origin/main` to scan only changed files on a PR. Add `--json` for machine-readable output.

## Config (optional)

Create `react-doctor.config.json` to suppress rules that don't apply:

```json
{
  "ignore": ["no-inline-styles"],
  "threshold": 70
}
```

## Verify

Run `npx react-doctor@latest` and confirm:
- [ ] A score is printed (0–100)
- [ ] Diagnostics list specific files and line numbers
- [ ] `--diff origin/main` returns only changed-file issues
