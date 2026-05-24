---
name: a11y
description: Audit and fix accessibility issues in any web app to WCAG 2.2 AA standard. Covers keyboard navigation, screen reader support, color contrast, focus management, and ARIA. Use when preparing an app for launch or fixing reported accessibility issues.
argument-hint: <target area or URL to audit>
---

# A11y

You are auditing and fixing accessibility issues to WCAG 2.2 AA. Work through each phase in order.

**Target:** {{args}}


## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx skills update a11y -y
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Phase 1: Automated Audit

Run automated tools first to find the easy wins:

1. Install axe-core: `bun add -d @axe-core/cli` or run via browser extension
2. Run against all key pages:
   ```bash
   bunx axe-cli <URL> --include main --reporter json > axe-report.json
   ```
3. Also run Lighthouse accessibility audit: score and findings
4. Document all violations with severity (critical, serious, moderate, minor)

Automated tools catch ~30% of issues. Manual testing is required for the rest.


## Phase 2: Manual Audit

Spawn **3 parallel subagents** to check different categories:

| Subagent | Categories |
|----------|-----------|
| 1 | **Keyboard navigation**: Tab order logical? All interactive elements focusable? No keyboard traps? Focus visible at all times? |
| 2 | **Semantic HTML & ARIA**: Headings hierarchical (h1→h2→h3)? Landmark regions present? Images have alt text? Forms have labels? Buttons have accessible names? |
| 3 | **Visual**: Color contrast ≥ 4.5:1 for normal text, ≥ 3:1 for large text? No information conveyed by color alone? Text resizes to 200% without breaking? |

Each subagent returns: specific violations with file locations and WCAG criteria violated.


## Phase 3: Prioritize

Classify all findings:

- **Critical (P0)**: Users with disabilities cannot complete core tasks: fix before launch
  - Form fields without labels
  - Images without alt text (if content-bearing)
  - Keyboard traps
  - Missing page `<title>`
- **Serious (P1)**: Significant barriers: fix in first sprint
  - Contrast failures on body text
  - Missing focus indicators
  - Interactive elements not keyboard accessible
- **Moderate (P2)**: Inconvenient but workaround exists: fix in first month
  - Missing skip navigation link
  - Inconsistent focus order
  - Missing ARIA labels on icon buttons
- **Minor (P3)**: Best practice improvements: backlog

Fix all P0 and P1 findings. Present P2 and P3 for user decision.


## Phase 4: Fix

For each finding, apply the standard fix:

**Missing label on input**:
```html
<label for="email">Email address</label>
<input id="email" type="email" />
<!-- or: aria-label="Email address" if no visible label -->
```

**Missing alt text**:
```html
<img src="..." alt="Description of image" />
<!-- decorative images: alt="" -->
```

**No focus indicator**:
```css
:focus-visible { outline: 2px solid var(--color-focus); outline-offset: 2px; }
/* Never use: outline: none without a replacement */
```

**Keyboard trap (modal)**:
- Trap focus inside modal when open using `focus-trap-react` or equivalent
- Return focus to trigger element when modal closes
- Close on Escape key

**Contrast failure**:
- Use [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- Adjust color values until ratio is ≥ 4.5:1 (normal text)

**Missing skip link**:
```html
<a href="#main-content" class="sr-only focus:not-sr-only">Skip to main content</a>
```


## Phase 5: Screen Reader Testing

Test with at least one screen reader:

- **Windows**: NVDA (free) + Chrome
- **macOS/iOS**: VoiceOver (built-in) + Safari
- **Test**: Navigate each key user flow using only the keyboard and screen reader

Verify:
- [ ] Page title announced on navigation
- [ ] Form labels read correctly
- [ ] Error messages announced when validation fails
- [ ] Modal open/close announced
- [ ] Dynamic content changes announced via `aria-live` regions


## Phase 6: Verify

Re-run automated audit after fixes:

- [ ] axe-core reports zero critical/serious violations
- [ ] Lighthouse accessibility score ≥ 90
- [ ] All interactive elements reachable by Tab key
- [ ] Focus visible on all interactive elements
- [ ] Core user flows completable with keyboard only


## Completion Report

- Automated findings: total count, fixed count, deferred count
- Manual findings by category
- P0/P1 fixes applied (list with WCAG criteria)
- Screen reader test results
- Lighthouse score before and after
