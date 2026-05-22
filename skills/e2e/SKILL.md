---
name: e2e
description: End-to-end test authoring and execution for any web or CLI app. Discovers user flows, sets up the E2E framework if needed, writes tests covering the golden path and critical edge cases, runs them, and fixes failures. Use when asked to write E2E tests, add Playwright/Cypress coverage, or verify a feature works from the user's perspective.
argument-hint: <feature, flow, or area to cover>
---

# e2e — End-to-End Test Authoring & Execution

You are writing end-to-end tests that simulate real user behavior. Work through each phase in order. Do not skip phases.

**Target:** {{args}}

---

## Phase 1: Discover

Spawn **3 parallel subagents** to map what needs testing:

| Subagent | Focus | What to find |
|----------|-------|-------------|
| 1 | **App entry points** | How the app starts, what port/URL it runs on, how to launch it in test mode, any seed/fixture scripts |
| 2 | **User flows** | Routes, pages, forms, actions, and navigation paths relevant to the target area |
| 3 | **Existing tests** | Current E2E framework (Playwright, Cypress, etc.), test structure, helper utilities, existing coverage gaps |

Synthesize into:
- **Launch command** — how to start the app under test
- **Base URL** — where to point the browser
- **Flow inventory** — numbered list of user flows to cover, with the UI elements involved
- **Framework decision** — use existing setup; if none exists, recommend Playwright (default) and confirm with user

---

## Phase 2: Framework Setup (if needed)

If no E2E framework is installed:

1. Install the framework: `bun add -d @playwright/test` (or equivalent)
2. Generate config: `bunx playwright install --with-deps chromium`
3. Create a minimal config file following the project's conventions
4. Add a test script to `package.json`: `"test:e2e": "playwright test"`
5. Verify the framework runs (empty test suite passes)

If a framework already exists, read its config and confirm the test directory and launch settings before proceeding.

---

## Phase 3: Plan Test Scenarios

For the target area, define test scenarios at two levels:

**Golden path** — the happy path a user follows when everything works:
- List each step as a user action (click, type, navigate, submit)
- Define the expected outcome after each step
- One scenario per major flow

**Critical edge cases** — the failures a user will actually hit:
- Empty states (no data, first use)
- Validation errors (bad input, required fields)
- Auth boundaries (logged out, wrong role)
- Error recovery (server error, network failure)
- Redirect and navigation correctness after actions

Present the scenario list to the user and confirm scope before writing any tests. Note which flows are already covered by existing tests.

---

## Phase 4: Write Tests

For each confirmed scenario:

1. Read similar existing tests first — match their style, helpers, and assertions exactly
2. Write the test using the Page Object pattern if the project already uses it; otherwise write direct locator calls
3. Use **semantic selectors** in priority order:
   - `getByRole` / `getByLabel` / `getByText` (prefer these — they match what users see)
   - `data-testid` attributes (add them to the source if needed)
   - CSS selectors only as a last resort
4. Assert the **visible outcome** the user would notice — page content, URL, toast message, element state
5. Never assert implementation details (Redux state, internal API calls, class names)

**Rules:**
- Each test must be independent — no shared state between tests
- Each test must clean up after itself or rely on fresh app state
- Flaky assertions (timing, animation) must use `waitFor` or explicit waits, never `sleep`
- If a `data-testid` is missing from the UI, add it to the source file in the same PR

Run the full test file immediately after writing it. A passing golden-path test is the baseline.

---

## Phase 5: Fix Failures

For every failing test:

1. Read the error output carefully — distinguish selector mismatch, timing issue, and actual behavior bug
2. **Selector mismatch** — fix the locator or add a `data-testid` to the source
3. **Timing issue** — replace fixed waits with `waitFor` targeting a visible element or network idle
4. **Actual bug found** — this is a real find. Report it to the user before deciding whether to fix the app or mark the test as known-failing with a TODO

Do not modify assertions to make tests pass — assertions represent the correct expected behavior.

Fix → run → fix → run. One failure at a time.

---

## Phase 6: Run Full Suite & Confirm

Run the complete E2E suite:

```
bun test:e2e
```

All tests must pass. If existing E2E tests are now failing, treat them as regressions and fix the cause before reporting done.

Report:
- How many flows are now covered
- How many tests were written (golden path vs. edge cases)
- Any bugs found in the app during testing
- Any flows intentionally deferred (with reason)

---

## Completion Checklist

- [ ] App launch and base URL confirmed
- [ ] E2E framework installed and configured
- [ ] Flow inventory created and confirmed with user
- [ ] Golden path test written and passing for each major flow
- [ ] Critical edge cases covered with dedicated tests
- [ ] All tests use semantic selectors or `data-testid`
- [ ] No flaky waits — all async assertions use `waitFor`
- [ ] Full E2E suite passes with no regressions
- [ ] Any app bugs found are reported to the user
