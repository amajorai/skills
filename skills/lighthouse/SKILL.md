---
name: lighthouse
description: Run Lighthouse audits on a website, analyze the results across all four categories (Performance, Accessibility, Best Practices, SEO), prioritize fixes by impact, implement them, and verify the scores improved. Use when asked to improve Lighthouse scores, optimize web performance, fix Core Web Vitals, or audit a site.
argument-hint: <URL or local dev server address>
---

# Lighthouse

You are auditing and optimizing a website for Lighthouse scores. Work through each phase in order. Do not skip phases.

**Target:** {{args}}

## Phase 0: Auto-Update

*Skip unless `{{args}}` contains `--update`, or `SKILLS_AUTO_UPDATE: true` is set in your project CLAUDE.md.*

```bash
npx --yes skills update amajorai/skills -y 2>/dev/null || true
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Phase 1: Setup & Baseline

If `{{args}}` is empty, ask for the target URL before running anything. If it's a local dev server, start it first.

Ensure Lighthouse is available: `npx lighthouse --version 2>/dev/null || bunx lighthouse --version 2>/dev/null`

Run the baseline audit:

```bash
npx lighthouse {{args}} --output=json --output-path=./lighthouse-baseline.json \
  --chrome-flags="--headless --no-sandbox" \
  --only-categories=performance,accessibility,best-practices,seo --quiet
```

Display scores (🔴 0–49, 🟡 50–89, 🟢 90–100) and Core Web Vitals:
LCP < 2.5s | INP < 200ms | CLS < 0.1 | FCP < 1.8s | TTFB < 800ms | TBT < 200ms

## Phase 2: Spawn Analysis Subagents

Spawn **4 parallel subagents**, each reading `./lighthouse-baseline.json`:

| Subagent | Category | What to extract |
|----------|----------|-----------------|
| 1 | Performance | Failed audits, opportunity savings (ms/KB), diagnostics sorted by impact |
| 2 | Accessibility | Failed audits, affected element counts, WCAG failure type |
| 3 | Best Practices | Failed audits, deprecation warnings, console errors, security issues |
| 4 | SEO | Failed audits, missing meta tags, crawlability issues, structured data problems |

## Phase 3: Prioritize & Plan

Consolidate findings. Assign Impact (High/Medium/Low), Effort (Low/Medium/High), and Category (P/A/B/S) to each issue. Present a ranked table and ask: **"Which issues should I fix? (all / just Performance / select numbers)"** Do not implement anything until the user answers.

## Phase 4: Implement Fixes

Work through selected fixes one at a time. For each fix:

1. Read the relevant source files first
2. Make the minimal surgical change
3. Run the dev server and confirm no regressions

**Performance fixes:** see [references/performance-fixes.md](references/performance-fixes.md)

**Accessibility fixes:** see [references/accessibility-fixes.md](references/accessibility-fixes.md)

**Best Practices & SEO fixes:** see [references/best-practices-seo-fixes.md](references/best-practices-seo-fixes.md)

## Phase 5: Re-audit & Compare

Re-run Lighthouse (same flags, `--output-path=./lighthouse-after.json`) and display a before/after table for all four categories plus Core Web Vitals.

If any score went **down**, investigate why and fix before reporting done.

## Phase 6: Verify in Browser

If running locally: open in browser, run DevTools Lighthouse to confirm scores match CLI results, verify the golden path (navigation, forms, key interactions), and check for visual regressions.

## Completion Checklist

- [ ] Baseline audit completed and scores recorded
- [ ] All four categories analyzed and findings prioritized
- [ ] Fix scope confirmed with user
- [ ] Images: `width`/`height` set, lazy loading added, LCP image prioritized
- [ ] Render-blocking JS deferred or made async
- [ ] Fonts: `font-display: swap`, preloaded if critical
- [ ] Static assets have long-lived `Cache-Control` headers
- [ ] All `<img>` elements have `alt` attributes
- [ ] Color contrast meets WCAG AA
- [ ] `<html lang="">` and `<title>` present on all pages
- [ ] `<meta name="description">` and viewport tag present
- [ ] `target="_blank"` links have `rel="noopener noreferrer"`
- [ ] Console errors resolved
- [ ] Re-audit completed: all scores equal or better than before
- [ ] No visual or functional regressions introduced
- [ ] Final scores reported with before/after comparison
