# expect Reference

## What it does

Reads your current `git diff`, generates a test plan covering the changes, and executes it in a real Playwright browser. No selectors or assertions to write: the agent figures it out from the diff.

Catches: broken hover states, missing links, dead buttons, performance regressions (LCP, animation frames), CSRF issues, missing metadata.

## Install

```bash
npm install -g expect-cli
```

## Run

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

## CI integration (optional)

```yaml
- name: Expect browser tests
  run: expect-cli --ci --agent claude
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Add to CLAUDE.md

```markdown
## Testing
After any UI change, run `/expect` to verify the change in a real browser.
Use `/expect -m "<focus area>"` to target a specific flow.
```

## Verify

Make a small visible change (e.g., change a button label), then run `/expect -y` and confirm:
- [ ] expect reads the git diff
- [ ] Generates a test plan referencing the changed component
- [ ] Opens a browser and runs through the plan
- [ ] Reports pass/fail per test step
