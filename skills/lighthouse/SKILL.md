---
name: lighthouse
description: Run Lighthouse audits on a website, analyze the results across all four categories (Performance, Accessibility, Best Practices, SEO), prioritize fixes by impact, implement them, and verify the scores improved. Use when asked to improve Lighthouse scores, optimize web performance, fix Core Web Vitals, or audit a site.
argument-hint: <URL or local dev server address>
---

# Lighthouse

You are auditing and optimizing a website for Lighthouse scores. Work through each phase in order. Do not skip phases.

**Target:** {{args}}


## Phase 0: Auto-Update

*Skip if `{{args}}` contains `--no-update`, or if `SKILLS_AUTO_UPDATE: false` is set in your project CLAUDE.md.*

```bash
npx skills update lighthouse -y
```

If the skill was updated, stop here and tell the user: **"This skill was just updated. Re-run your command to use the new version."** Otherwise continue silently.

## Phase 1: Setup & Baseline

First, ensure Lighthouse is available:

```bash
# Check if lighthouse CLI is available
npx lighthouse --version 2>/dev/null || bunx lighthouse --version 2>/dev/null

# If not installed globally, use via npx (no install needed)
# If the project uses bun: bunx @lhci/cli --version
```

If the target is a local dev server, start it first and confirm it's running before proceeding.

Run the baseline audit across all four categories:

```bash
npx lighthouse {{args}} \
  --output=json \
  --output-path=./lighthouse-baseline.json \
  --chrome-flags="--headless --no-sandbox" \
  --only-categories=performance,accessibility,best-practices,seo \
  --quiet
```

If `{{args}}` is empty, ask the user for the URL before proceeding.

Parse and display the baseline scores in a table:

| Category | Score | Status |
|----------|-------|--------|
| Performance | X | 🔴/🟡/🟢 |
| Accessibility | X | 🔴/🟡/🟢 |
| Best Practices | X | 🔴/🟡/🟢 |
| SEO | X | 🔴/🟡/🟢 |

Score legend: 🔴 0–49, 🟡 50–89, 🟢 90–100

Also extract and show Core Web Vitals:
- **LCP** (Largest Contentful Paint): target < 2.5s
- **INP** (Interaction to Next Paint): target < 200ms
- **CLS** (Cumulative Layout Shift): target < 0.1
- **FCP** (First Contentful Paint): target < 1.8s
- **TTFB** (Time to First Byte): target < 800ms
- **TBT** (Total Blocking Time): target < 200ms


## Phase 2: Spawn Analysis Subagents

Spawn **4 parallel subagents** to read the baseline JSON and identify issues per category:

| Subagent | Category | What to extract |
|----------|----------|-----------------|
| 1 | **Performance** | Failed audits, opportunity savings (ms/KB), diagnostics: sorted by estimated impact |
| 2 | **Accessibility** | Failed audits, affected element counts, WCAG failure type |
| 3 | **Best Practices** | Failed audits, deprecation warnings, console errors, security issues |
| 4 | **SEO** | Failed audits, missing meta tags, crawlability issues, structured data problems |

Each subagent reads `./lighthouse-baseline.json` and returns a prioritized list of findings.


## Phase 3: Prioritize & Plan

Consolidate all findings. For each issue, assign:
- **Impact** (High / Medium / Low): based on Lighthouse's estimated savings or score weight
- **Effort** (Low / Medium / High): based on how invasive the fix is
- **Category**: P (Performance), A (Accessibility), B (Best Practices), S (SEO)

Present a ranked table to the user:

| # | Issue | Category | Impact | Effort | Fix |
|---|-------|----------|--------|--------|-----|
| 1 | Eliminate render-blocking resources | P | High | Medium | Defer/async non-critical JS/CSS |
| 2 | Properly size images | P | High | Low | Add `width`/`height`, use `srcset` |
| ... | | | | | |

Ask the user: **"Which issues should I fix? (all / just Performance / select numbers)"**

Do not implement anything until the user answers.


## Phase 4: Implement Fixes

Work through selected fixes one at a time. For each fix:

1. Read the relevant source files first
2. Make the minimal surgical change
3. Run the dev server and confirm no regressions

### Performance Fixes

**Images**
- Add explicit `width` and `height` to all `<img>` tags to prevent CLS
- Add `loading="lazy"` to below-the-fold images
- Add `fetchpriority="high"` to the LCP image
- Convert large PNGs/JPGs to WebP/AVIF using `<picture>` with fallback
- Use `srcset` for responsive images
- Compress images if oversized (use `sharp` CLI or equivalent)

**JavaScript**
- Move non-critical `<script>` tags to `defer` or `async`
- Identify and remove unused JS: check Lighthouse "Reduce unused JavaScript" audit
- Split large bundles: check if the framework supports dynamic `import()`
- Add `rel="modulepreload"` for critical JS modules

**CSS**
- Move `<link rel="stylesheet">` for non-critical CSS to load asynchronously:
  ```html
  <link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
  ```
- Remove unused CSS if the audit flags it
- Inline critical (above-the-fold) CSS in `<head>` for the fastest FCP

**Fonts**
- Add `rel="preconnect"` for Google Fonts or CDN font hosts
- Add `font-display: swap` to all `@font-face` declarations
- Preload the primary font file:
  ```html
  <link rel="preload" href="/fonts/main.woff2" as="font" type="font/woff2" crossorigin>
  ```

**Caching**
- Add `Cache-Control` headers for static assets (images, JS, CSS): `max-age=31536000, immutable`
- If using a framework with content hashing in filenames, enable long-lived caching
- Add `ETag` or `Last-Modified` for HTML responses

**Server / TTFB**
- Check if responses are gzip/brotli compressed: add if missing
- Add `rel="dns-prefetch"` or `rel="preconnect"` for third-party origins
- If server-side: check for slow database queries or missing indexes
- Consider adding a CDN or edge caching layer if TTFB is consistently > 800ms

**Third-party scripts**
- Load analytics, chat widgets, and ad scripts with `defer` or move to `async`
- Use Partytown or similar to offload heavy third-party scripts to a web worker

### Accessibility Fixes

**Images**
- Add descriptive `alt` text to all informative `<img>` elements
- Add `alt=""` to decorative images (not `alt` missing)
- Add `aria-label` or `aria-labelledby` to `<svg>` icons used as buttons

**Color contrast**
- Check Lighthouse's contrast ratio findings
- Increase foreground/background contrast to meet WCAG AA (4.5:1 for normal text, 3:1 for large text)
- Never fix by removing the color: fix the specific hex values

**Keyboard navigation**
- Ensure all interactive elements are reachable via Tab
- Add `tabindex="0"` to custom interactive elements (divs/spans acting as buttons)
- Never use `tabindex="-1"` on visible interactive elements unless intentional
- Ensure focus indicators are visible (don't suppress `:focus` outline without a replacement)

**ARIA**
- Add missing `aria-label` to icon buttons and inputs without visible labels
- Ensure `role` attributes are valid and match the element's purpose
- Add `aria-expanded`, `aria-controls` to disclosure widgets (accordions, menus)
- Ensure `<form>` inputs have associated `<label>` elements

**Landmarks and headings**
- Wrap main content in `<main>`
- Ensure heading hierarchy is sequential (h1 → h2 → h3, no skipping levels)
- Add `<nav>` around navigation lists

**Documents**
- Add `lang` attribute to `<html>` element: `<html lang="en">`
- Ensure `<title>` is present and descriptive on every page

### Best Practices Fixes

**Security**
- Ensure the site is served over HTTPS
- Add `rel="noopener noreferrer"` to `target="_blank"` links
- Add a Content Security Policy header (start with `report-only` mode)
- Add `X-Content-Type-Options: nosniff` and `X-Frame-Options: SAMEORIGIN` headers

**Console errors**
- Open the Lighthouse JSON and find any logged console errors
- Fix the underlying JS errors: never suppress them silently

**Deprecated APIs**
- Update any APIs flagged as deprecated (check the audit's `items` for specifics)

**Doctype**
- Ensure `<!DOCTYPE html>` is the first line of every HTML page

### SEO Fixes

**Meta tags**
- Add `<meta name="description" content="...">` if missing (150–160 chars)
- Add `<title>` if missing or generic
- Add Open Graph tags for social sharing:
  ```html
  <meta property="og:title" content="...">
  <meta property="og:description" content="...">
  <meta property="og:image" content="...">
  <meta property="og:url" content="...">
  ```

**Crawlability**
- Ensure `<meta name="robots" content="noindex">` is NOT present on public pages
- Add a `robots.txt` file if missing:
  ```
  User-agent: *
  Allow: /
  Sitemap: https://example.com/sitemap.xml
  ```
- Add an XML sitemap and link it from `robots.txt`
- Ensure links use descriptive text (no "click here" or "read more")

**Structured data**
- Add JSON-LD schema markup for the page type (Article, Product, Organization, etc.)
- Validate with Google's Rich Results Test

**Mobile**
- Ensure `<meta name="viewport" content="width=device-width, initial-scale=1">` is present
- Confirm tap targets are at least 48×48px with 8px spacing


## Phase 5: Re-audit & Compare

After all selected fixes are implemented, re-run Lighthouse:

```bash
npx lighthouse {{args}} \
  --output=json \
  --output-path=./lighthouse-after.json \
  --chrome-flags="--headless --no-sandbox" \
  --only-categories=performance,accessibility,best-practices,seo \
  --quiet
```

Display a before/after comparison:

| Category | Before | After | Delta |
|----------|--------|-------|-------|
| Performance | X | X | +/- X |
| Accessibility | X | X | +/- X |
| Best Practices | X | X | +/- X |
| SEO | X | X | +/- X |

Also compare Core Web Vitals before/after.

If any score went **down**, investigate why and fix before reporting done.


## Phase 6: Verify in Browser

If the app is running locally:

1. Open the target URL in the browser
2. Run Chrome DevTools Lighthouse (or use the Lighthouse CLI with `--view`) to confirm the scores match the CLI results
3. Manually verify the golden path still works (navigation, forms, key interactions)
4. Check for any visual regressions caused by the changes


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
