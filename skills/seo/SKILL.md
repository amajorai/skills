---
name: seo
description: Audit and optimize a web project for SEO. Detects the stack, runs a structured interview, then implements only the selected improvements: meta tags, structured data, sitemaps, robots.txt, page speed, Core Web Vitals, and more.
argument-hint: [URL or project path, optional]
---

# SEO

You are optimizing a web project for search engine visibility and ranking. **Do not change anything until the interview is complete and the user has confirmed their priorities.** Order matters: detect → interview → plan → implement.

**Target:** {{args}}


## Phase 1: Detect Current State

Scan silently before asking anything. Adapt commands to the detected stack.

These detection, audit, and verification commands are bash (`grep -r`, `head`, `awk`, `ls -la`, `curl`, `2>/dev/null`). On Windows, run them via the Bash tool or Git Bash, since they fail in PowerShell.

```bash
ls package.json next.config.* astro.config.* nuxt.config.* vite.config.* gatsby-config.* remix.config.* 2>/dev/null | head -20
cat package.json 2>/dev/null | grep -E '"next"|"astro"|"nuxt"|"gatsby"|"remix"|"vite"|"react"|"vue"|"svelte"' | head -10
ls -la robots.txt sitemap.xml sitemap*.xml public/robots.txt public/sitemap*.xml 2>/dev/null
grep -r "og:" src/ app/ pages/ --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.astro" --include="*.vue" --include="*.svelte" -l 2>/dev/null | head -5
grep -r "<title" src/ app/ pages/ --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.astro" --include="*.vue" -l 2>/dev/null | head -5
grep -r "application/ld+json" src/ app/ pages/ -l 2>/dev/null | head -5
grep -rn "<img " src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" 2>/dev/null | grep -v "alt=" | head -10
grep -r "canonical" src/ app/ pages/ -l 2>/dev/null | head -5
grep -r "font-display" src/ styles/ --include="*.css" --include="*.scss" -l 2>/dev/null | head -3
grep -r "hreflang" src/ -l 2>/dev/null | head -3
```

Note: framework/renderer (SSR/SSG/SPA), meta tag presence, sitemap/robots.txt, structured data, images missing alt, i18n.


## Phase 2: Full Interview

Present everything in one message. Flag detected gaps as ⚠️.

> **SEO optimization setup: tell me what you want and I'll implement it all in one pass.**
>
> I've scanned your project. Here's what I found:
> - **Framework / rendering:** [Next.js SSR / Astro SSG / SPA / etc.]
> - **Meta tags:** [present / missing on N pages]
> - **Open Graph:** [present / missing]
> - **Sitemap:** [found at path / not found ⚠️]
> - **robots.txt:** [found / not found ⚠️]
> - **Structured data (JSON-LD):** [present / absent]
> - **Images without alt text:** [count or "none found"]
> - **Canonical tags:** [present / missing]
> - **i18n / hreflang:** [detected / not detected]
>
> **What is this site?** (name, primary language, target region, goal: blog/e-commerce/SaaS/portfolio/docs/other, pages to exclude from indexing)
>
> ---
>
> **A. Meta tags & Open Graph**
> - [ ] `<title>` tags: unique, ≤60 chars, keyword-first
> - [ ] `<meta name="description">`: 120–160 chars, CTA-oriented
> - [ ] Open Graph (`og:title`, `og:description`, `og:image`, `og:url`)
> - [ ] Twitter Card tags
> → Default OG image URL?
>
> **B. Sitemap**
> - [ ] Generate `sitemap.xml`
> - [ ] Submit sitemap to Google Search Console
>
> **C. robots.txt**
> - [ ] Create/update `robots.txt` → Paths to disallow?
>
> **D. Canonical Tags**
> - [ ] Add `<link rel="canonical">` to prevent duplicate content
>
> **E. Structured Data (JSON-LD)** — select applicable types:
> - [ ] `WebSite` / `Organization` / `Person`
> - [ ] `Article` / `BlogPosting`
> - [ ] `Product`
> - [ ] `BreadcrumbList`
> - [ ] `FAQPage`
> - [ ] `LocalBusiness`
>
> **F. Image Optimization**
> - [ ] Add missing `alt` attributes
> - [ ] Add `width`/`height` to prevent CLS
> - [ ] Switch to framework image component (`next/image`, `@astrojs/image`, etc.)
> - [ ] Add `loading="lazy"` to below-the-fold images
>
> **G. Performance (Core Web Vitals)**
> - [ ] `font-display: swap`
> - [ ] Preconnect to external font/CDN origins
> - [ ] Preload above-the-fold hero image
> - [ ] `fetchpriority="high"` on LCP image
> - [ ] Audit render-blocking scripts (defer/async)
>
> **H. hreflang** *(only shown if i18n detected)*
> - [ ] `<link rel="alternate" hreflang="...">` for each locale + `x-default`
>
> **I. Heading Hierarchy**
> - [ ] Audit `<h1>`–`<h6>`: one `<h1>` per page, logical nesting
>
> **J. Internal Linking**
> - [ ] Audit orphan pages (no inbound internal links)
> - [ ] Ensure descriptive link text (no "click here")
>
> **K. robots meta / noindex**
> - [ ] Add `noindex,nofollow` to non-indexable pages

Wait for answers. Once confirmed, ask: **"Ready to proceed?"**


## Phase 3: Pre-flight Notes

- **Meta tags on SSR/SSG:** confirm where `<head>` is managed (layout file, `_document.tsx`, `<Head>`, react-helmet, next/head, Astro head).
- **Sitemap:** static (build-time) or dynamic (runtime)? Dynamic sites with a CMS may need ISR or scheduled regeneration.
- **Structured data for Product:** confirm price currency and whether reviews are available — incomplete Product schema triggers Google rich result errors.
- **hreflang:** confirm all locale URLs exist and return 200 — broken hreflang confuses Google's locale detection.


## Phase 4: Meta Tags & Open Graph (if selected: A)

Adapt to the detected framework. See [references/meta-tags.md](references/meta-tags.md) for framework-specific code templates (Next.js App Router, Pages Router, Astro, plain HTML).

Audit all page templates: every page needs a unique title and description. Identical titles across pages are a ranking signal problem.


## Phase 5: Sitemap (if selected: B)

See [references/sitemap-templates.md](references/sitemap-templates.md) for code (Next.js App Router/Pages Router, Astro, static XML).

After generating, add the sitemap URL to `robots.txt`:
```
Sitemap: https://example.com/sitemap.xml
```


## Phase 6: robots.txt (if selected: C)

```
# public/robots.txt
User-agent: *
Allow: /

Disallow: /admin/
Disallow: /api/
# add any other paths the user selected

Sitemap: https://example.com/sitemap.xml
```

Place in `public/robots.txt` for Next.js/Astro/Vite, or at the server root. Verify accessible at `https://example.com/robots.txt`.


## Phase 7: Structured Data (if selected: E)

Inject via `<script type="application/ld+json">` in the page `<head>` using the framework-appropriate mechanism. See [references/structured-data.md](references/structured-data.md) for all schema templates (WebSite, Organization, Article/BlogPosting, Product, FAQPage, BreadcrumbList).

Validate all structured data with Google's Rich Results Test.


## Phase 8: Image Optimization (if selected: F)

For each `<img>` without `alt`: derive a descriptive alt from context. Use `alt=""` for decorative images only. Never use the filename as alt text.

```html
<!-- Hero / LCP image -->
<img src="/hero.jpg" alt="Team collaborating around a whiteboard" width="1200" height="630" loading="eager" fetchpriority="high" />

<!-- Below the fold -->
<img src="/feature.jpg" alt="Feature description" width="600" height="400" loading="lazy" />
```

For Next.js, replace `<img>` with `<Image>` from `next/image` and add `priority` to the LCP image.


## Phase 9: Performance / Core Web Vitals (if selected: G)

```html
<!-- Preconnect to external origins -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />

<!-- Preload hero/LCP image -->
<link rel="preload" as="image" href="/hero.jpg" fetchpriority="high" />
```

```css
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter.woff2') format('woff2');
  font-display: swap;
}
```

```html
<script src="/analytics.js" defer></script>
<script src="/chat-widget.js" async></script>
```


## Phase 10: hreflang (if selected: H)

```html
<head>
  <link rel="alternate" hreflang="en" href="https://example.com/en/page" />
  <link rel="alternate" hreflang="es" href="https://example.com/es/page" />
  <link rel="alternate" hreflang="x-default" href="https://example.com/en/page" />
</head>
```

Rules: every localized page must include hreflang for ALL locales (including itself). `x-default` points to the fallback locale. Each alternate URL must return a 200 — no redirects.

For Next.js App Router:
```tsx
export const metadata: Metadata = {
  alternates: {
    canonical: 'https://example.com/en/page',
    languages: {
      'en-US': 'https://example.com/en/page',
      'es-ES': 'https://example.com/es/page',
    },
  },
}
```


## Phase 11: Heading Audit (if selected: I)

```bash
grep -rL "<h1" src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" --include="*.vue" 2>/dev/null
grep -rn "<h1" src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" 2>/dev/null | \
  awk -F: '{print $1}' | sort | uniq -d
```

Fix: each page has exactly one `<h1>` containing the primary keyword. Subheadings in logical order, no skipped levels.


## Phase 12: noindex Tags (if selected: K)

```html
<meta name="robots" content="noindex, nofollow" />
```

Common candidates: `/thank-you`, `/order-confirmation`, `/login`, `/signup`, `/dashboard`, paginated pages beyond page 2, tag/category archives with thin content.

For Next.js App Router:
```tsx
export const metadata: Metadata = {
  robots: { index: false, follow: false },
}
```


## Phase 13: Final Verification

See [references/completion-checklist.md](references/completion-checklist.md) for the full verification command block and completion checklist.

Key live-URL checks (requires deployed site):
```bash
curl -s https://example.com/sitemap.xml | head -20
curl -s https://example.com/robots.txt
curl -s https://example.com | grep -E "<title|og:title|description" | head -10
```
