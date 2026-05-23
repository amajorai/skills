---
name: seo
description: Audit and optimize a web project for SEO. Detects the stack, runs a structured interview, then implements only the selected improvements: meta tags, structured data, sitemaps, robots.txt, page speed, Core Web Vitals, and more.
argument-hint: [URL or project path, optional]
---

# SEO

You are optimizing a web project for search engine visibility and ranking. **Do not change anything until the interview is complete and the user has confirmed their priorities.** Order matters: detect → interview → plan → implement.

**Target:** {{args}}


## Phase 0: Detect Current State

Scan silently before asking anything. Adapt commands to the detected stack.

```bash
# Identify framework / stack
ls package.json next.config.* astro.config.* nuxt.config.* vite.config.* gatsby-config.* remix.config.* 2>/dev/null | head -20
cat package.json 2>/dev/null | grep -E '"next"|"astro"|"nuxt"|"gatsby"|"remix"|"vite"|"react"|"vue"|"svelte"' | head -10

# Existing SEO files
ls -la robots.txt sitemap.xml sitemap*.xml public/robots.txt public/sitemap*.xml 2>/dev/null

# Meta tags: sample a few pages
grep -r "og:" src/ app/ pages/ --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.astro" --include="*.vue" --include="*.svelte" -l 2>/dev/null | head -5
grep -r "<title" src/ app/ pages/ --include="*.html" --include="*.tsx" --include="*.jsx" --include="*.astro" --include="*.vue" -l 2>/dev/null | head -5
grep -r "description" src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" -l 2>/dev/null | head -5

# Structured data
grep -r "application/ld+json" src/ app/ pages/ -l 2>/dev/null | head -5

# Images: check for alt text and next/image usage
grep -rn "<img " src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" 2>/dev/null | grep -v "alt=" | head -10

# Canonical tags
grep -r "canonical" src/ app/ pages/ -l 2>/dev/null | head -5

# Performance hints
grep -r "font-display" src/ styles/ --include="*.css" --include="*.scss" -l 2>/dev/null | head -3
grep -r "loading=" src/ app/ pages/ --include="*.tsx" --include="*.jsx" -l 2>/dev/null | head -5

# Existing sitemap generator
grep -r "sitemap" package.json src/ 2>/dev/null | head -5

# Internationalization
ls src/i18n src/locales public/locales 2>/dev/null
grep -r "hreflang" src/ -l 2>/dev/null | head -3
```

Analyze and note:
- What framework / renderer (SSR, SSG, SPA, MPA)?
- Are meta tags and Open Graph present?
- Is there a sitemap and robots.txt?
- Any structured data (JSON-LD)?
- Images missing `alt` attributes?
- Is the site internationalized?


## Phase 1: Full Interview

Present everything in one message. Tailor the checklist based on what you detected in Phase 0. Flag any detected gaps as ⚠️.


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
> ---
>
> **What is this site?**
> - Site name, primary language, and target region
> - Primary goal: blog / e-commerce / SaaS / portfolio / docs / other
> - Any pages to exclude from indexing?
>
> ---
>
> **Select what you want to set up:**
>
> **A. Meta tags & Open Graph**
> *(Flag if missing or incomplete)*
> - [ ] Add/fix `<title>` tags: unique, ≤60 chars, keyword-first
> - [ ] Add/fix `<meta name="description">`: 120–160 chars, CTA-oriented
> - [ ] Add/fix Open Graph (`og:title`, `og:description`, `og:image`, `og:url`)
> - [ ] Add Twitter Card tags (`twitter:card`, `twitter:title`, `twitter:image`)
> → If yes: **What is the default OG image URL?** (e.g. `/og-image.png`)
>
> **B. Sitemap**
> *(Flag if not found)*
> - [ ] Generate `sitemap.xml`: lists all indexable URLs
> → For dynamic sites: generate programmatically from routes/pages
> → For static sites: use a build-time plugin or script
> - [ ] Submit sitemap URL to Google Search Console (provide instructions)
>
> **C. robots.txt**
> *(Flag if not found)*
> - [ ] Create/update `robots.txt`
> → Any paths to disallow? (e.g. `/admin`, `/api`, `/checkout`)
>
> **D. Canonical Tags**
> - [ ] Add `<link rel="canonical">` to prevent duplicate content
>
> **E. Structured Data (JSON-LD)**
> Select schema types relevant to your site:
> - [ ] `WebSite`: site name, search action
> - [ ] `Organization` / `Person`: brand identity, social profiles
> - [ ] `Article` / `BlogPosting`: for blog posts
> - [ ] `Product`: for e-commerce (name, price, availability, reviews)
> - [ ] `BreadcrumbList`: breadcrumb navigation
> - [ ] `FAQPage`: FAQ sections
> - [ ] `LocalBusiness`: address, hours, phone (local SEO)
>
> **F. Image Optimization**
> - [ ] Add missing `alt` attributes (descriptive, keyword-relevant)
> - [ ] Add `width` and `height` attributes to prevent layout shift (CLS)
> - [ ] Switch `<img>` to framework image component where available (e.g. `next/image`, `@astrojs/image`)
> - [ ] Add `loading="lazy"` to below-the-fold images
>
> **G. Performance (Core Web Vitals)**
> Core Web Vitals directly affect Google rankings.
> - [ ] Add `font-display: swap` to prevent invisible text during font load (FCP)
> - [ ] Preconnect to external font/CDN origins (`<link rel="preconnect">`)
> - [ ] Preload above-the-fold hero image (`<link rel="preload" as="image">`)
> - [ ] Add `fetchpriority="high"` to the LCP image
> - [ ] Audit and reduce render-blocking scripts (defer/async)
>
> **H. Internationalization (hreflang)**
> *(Only shown if i18n detected)*
> - [ ] Add `<link rel="alternate" hreflang="...">` tags for each language/region
> - [ ] Add `x-default` hreflang for the default locale
>
> **I. Heading Hierarchy**
> - [ ] Audit `<h1>`–`<h6>` structure: ensure exactly one `<h1>` per page, logical nesting
>
> **J. Internal Linking**
> - [ ] Audit pages with no inbound internal links (orphan pages)
> - [ ] Ensure link text is descriptive (no "click here" or "read more")
>
> **K. robots meta / noindex**
> - [ ] Add `<meta name="robots" content="noindex,nofollow">` to pages that should not be indexed (e.g. thank-you pages, login pages, paginated duplicates)

Wait for the user's answers. Once confirmed, summarize the plan and ask: **"Ready to proceed?"**


## Phase 2: Pre-flight Notes

Before writing anything, flag these based on the user's selections:

**If meta tags on an SSR/SSG framework:** Confirm where `<head>` is managed: layout file, `_document.tsx`, `<Head>` component, or a meta framework like `react-helmet` / `next/head` / Astro's `<head>`.

**If sitemap generation:** Confirm whether it should be static (generated at build time) or dynamic (served at runtime). For dynamic sites with a CMS, a build-time approach may miss new content: recommend a scheduled regeneration or ISR.

**If structured data for Product:** Confirm price currency and whether reviews are available: incomplete Product schema can trigger Google rich result errors.

**If hreflang:** Confirm all locale URLs exist and are accessible: broken hreflang tags can confuse Google's locale detection.


## Phase 3: Meta Tags & Open Graph (if selected: A)

Adapt to the detected framework.

### Next.js App Router
```tsx
// app/layout.tsx or per-page page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: {
    template: '%s | Site Name',
    default: 'Site Name: Tagline',
  },
  description: '120–160 char description with primary keyword.',
  openGraph: {
    title: 'Site Name: Tagline',
    description: '120–160 char description.',
    url: 'https://example.com',
    siteName: 'Site Name',
    images: [{ url: 'https://example.com/og-image.png', width: 1200, height: 630 }],
    locale: 'en_US',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Site Name: Tagline',
    description: '120–160 char description.',
    images: ['https://example.com/og-image.png'],
  },
  alternates: {
    canonical: 'https://example.com',
  },
}
```

### Next.js Pages Router
```tsx
// pages/_app.tsx or per-page
import Head from 'next/head'

<Head>
  <title>Page Title | Site Name</title>
  <meta name="description" content="120–160 char description." />
  <meta property="og:title" content="Page Title | Site Name" />
  <meta property="og:description" content="120–160 char description." />
  <meta property="og:image" content="https://example.com/og-image.png" />
  <meta property="og:url" content="https://example.com/page" />
  <meta name="twitter:card" content="summary_large_image" />
  <link rel="canonical" href="https://example.com/page" />
</Head>
```

### Astro
```astro
// src/layouts/BaseLayout.astro
const { title, description, image = '/og-image.png', canonicalURL } = Astro.props
<head>
  <title>{title} | Site Name</title>
  <meta name="description" content={description} />
  <meta property="og:title" content={`${title} | Site Name`} />
  <meta property="og:description" content={description} />
  <meta property="og:image" content={new URL(image, Astro.url)} />
  <meta property="og:url" content={canonicalURL ?? Astro.url} />
  <meta name="twitter:card" content="summary_large_image" />
  <link rel="canonical" href={canonicalURL ?? Astro.url} />
</head>
```

### Plain HTML / other frameworks
```html
<head>
  <title>Page Title | Site Name</title>
  <meta name="description" content="120–160 char description." />
  <meta property="og:title" content="Page Title | Site Name" />
  <meta property="og:description" content="120–160 char description." />
  <meta property="og:image" content="https://example.com/og-image.png" />
  <meta property="og:url" content="https://example.com/page" />
  <meta name="twitter:card" content="summary_large_image" />
  <link rel="canonical" href="https://example.com/page" />
</head>
```

Audit all page templates and ensure every page has unique title and description. Identical titles across pages are a ranking signal problem.


## Phase 4: Sitemap (if selected: B)

### Next.js App Router: `app/sitemap.ts`
```ts
import { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      changeFrequency: 'yearly',
      priority: 1,
    },
    {
      url: 'https://example.com/blog',
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.8,
    },
    // Add dynamic pages by fetching from CMS/DB
  ]
}
```

### Next.js Pages Router: `pages/sitemap.xml.tsx`
```tsx
import { GetServerSideProps } from 'next'

function Sitemap() { return null }

export const getServerSideProps: GetServerSideProps = async ({ res }) => {
  const pages = ['', '/about', '/blog']
  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${pages.map(path => `  <url>
    <loc>https://example.com${path}</loc>
    <lastmod>${new Date().toISOString()}</lastmod>
    <changefreq>weekly</changefreq>
    <priority>${path === '' ? '1.0' : '0.8'}</priority>
  </url>`).join('\n')}
</urlset>`

  res.setHeader('Content-Type', 'text/xml')
  res.write(sitemap)
  res.end()
  return { props: {} }
}

export default Sitemap
```

### Astro: `src/pages/sitemap.xml.ts`
```ts
import type { APIRoute } from 'astro'

export const GET: APIRoute = async () => {
  const pages = ['', '/about', '/blog']
  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${pages.map(path => `  <url>
    <loc>https://example.com${path}</loc>
    <changefreq>weekly</changefreq>
    <priority>${path === '' ? '1.0' : '0.8'}</priority>
  </url>`).join('\n')}
</urlset>`

  return new Response(sitemap, {
    headers: { 'Content-Type': 'application/xml' },
  })
}
```

### Static sites / generic
```xml
<!-- public/sitemap.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/</loc>
    <lastmod>YYYY-MM-DD</lastmod>
    <changefreq>monthly</changefreq>
    <priority>1.0</priority>
  </url>
</urlset>
```

After generating, add the sitemap URL to `robots.txt`:
```
Sitemap: https://example.com/sitemap.xml
```


## Phase 5: robots.txt (if selected: C)

```
# public/robots.txt
User-agent: *
Allow: /

# Disallow paths the user selected
Disallow: /admin/
Disallow: /api/
Disallow: /checkout/

Sitemap: https://example.com/sitemap.xml
```

Place in `public/robots.txt` for Next.js/Astro/Vite projects, or at the server root.

Verify it is accessible at `https://example.com/robots.txt` before declaring done.


## Phase 6: Structured Data (if selected: E)

Inject via `<script type="application/ld+json">`. For frameworks: inject in the page `<head>` using the appropriate mechanism (Next.js `metadata` / `Script`, Astro `<head>`, etc.).

### WebSite
```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "Site Name",
  "url": "https://example.com",
  "potentialAction": {
    "@type": "SearchAction",
    "target": "https://example.com/search?q={search_term_string}",
    "query-input": "required name=search_term_string"
  }
}
```

### Organization
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Company Name",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png",
  "sameAs": [
    "https://twitter.com/handle",
    "https://linkedin.com/company/name"
  ]
}
```

### Article / BlogPosting
```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Article Title",
  "description": "Article description.",
  "image": "https://example.com/article-image.jpg",
  "author": {
    "@type": "Person",
    "name": "Author Name"
  },
  "publisher": {
    "@type": "Organization",
    "name": "Site Name",
    "logo": { "@type": "ImageObject", "url": "https://example.com/logo.png" }
  },
  "datePublished": "2024-01-01",
  "dateModified": "2024-01-15"
}
```

### Product
```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Product Name",
  "image": "https://example.com/product.jpg",
  "description": "Product description.",
  "brand": { "@type": "Brand", "name": "Brand Name" },
  "offers": {
    "@type": "Offer",
    "price": "49.99",
    "priceCurrency": "USD",
    "availability": "https://schema.org/InStock",
    "url": "https://example.com/product"
  }
}
```

### FAQPage
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is X?",
      "acceptedAnswer": { "@type": "Answer", "text": "X is ..." }
    }
  ]
}
```

### BreadcrumbList
```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://example.com" },
    { "@type": "ListItem", "position": 2, "name": "Blog", "item": "https://example.com/blog" },
    { "@type": "ListItem", "position": 3, "name": "Article Title" }
  ]
}
```

Validate all structured data with Google's Rich Results Test: https://search.google.com/test/rich-results


## Phase 7: Image Optimization (if selected: F)

For each `<img>` without `alt`:
- Derive a descriptive alt from surrounding context or filename.
- For decorative images: `alt=""`.
- Never use filename as alt text (e.g. `alt="img_1234.jpg"`).

```html
<!-- Before -->
<img src="/hero.jpg" />

<!-- After -->
<img src="/hero.jpg" alt="Team collaborating around a whiteboard" width="1200" height="630" loading="eager" fetchpriority="high" />
```

For below-the-fold images:
```html
<img src="/feature.jpg" alt="Feature description" width="600" height="400" loading="lazy" />
```

For Next.js, replace `<img>` with `<Image>`:
```tsx
import Image from 'next/image'

<Image
  src="/hero.jpg"
  alt="Team collaborating around a whiteboard"
  width={1200}
  height={630}
  priority  // for LCP image
/>
```


## Phase 8: Performance / Core Web Vitals (if selected: G)

```html
<!-- Preconnect to external origins -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />

<!-- Preload hero/LCP image -->
<link rel="preload" as="image" href="/hero.jpg" fetchpriority="high" />
```

```css
/* font-display: swap to prevent FOIT */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter.woff2') format('woff2');
  font-display: swap;
}
```

```html
<!-- Defer non-critical scripts -->
<script src="/analytics.js" defer></script>
<!-- Or async for independent scripts -->
<script src="/chat-widget.js" async></script>
```


## Phase 9: hreflang (if selected: H)

```html
<head>
  <link rel="alternate" hreflang="en" href="https://example.com/en/page" />
  <link rel="alternate" hreflang="es" href="https://example.com/es/page" />
  <link rel="alternate" hreflang="x-default" href="https://example.com/en/page" />
</head>
```

Rules:
- Every localized page must include `hreflang` for ALL locales (including itself).
- `x-default` points to the default/fallback locale.
- Each alternate URL must return a 200: no redirects.

For Next.js App Router, use `alternates.languages` in metadata:
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


## Phase 10: Heading Audit (if selected: I)

```bash
# Pages with no h1
grep -rL "<h1" src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" --include="*.vue" 2>/dev/null

# Pages with multiple h1s
grep -rn "<h1" src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" 2>/dev/null | \
  awk -F: '{print $1}' | sort | uniq -d
```

Fix: ensure each page has exactly one `<h1>` that contains the primary keyword. Subheadings use `<h2>`–`<h6>` in logical order without skipping levels.


## Phase 11: noindex Tags (if selected: K)

```html
<meta name="robots" content="noindex, nofollow" />
```

Common candidates:
- `/thank-you`, `/order-confirmation`: no search value, creates duplicate intent
- `/login`, `/signup`, `/dashboard`: gated content
- Paginated pages beyond page 2 (`?page=3`, etc.): thin content
- Tag/category archive pages with little unique content

For Next.js App Router:
```tsx
export const metadata: Metadata = {
  robots: { index: false, follow: false },
}
```


## Phase 12: Final Verification

```bash
# Sitemap accessible
curl -s https://example.com/sitemap.xml | head -20

# robots.txt accessible
curl -s https://example.com/robots.txt

# Meta tags present on homepage
curl -s https://example.com | grep -E "<title|og:title|description" | head -10

# Images without alt (quick re-scan)
grep -rn "<img " src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" 2>/dev/null | grep -v "alt=" | head -10

# Check for multiple h1s
grep -rn "<h1" src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" 2>/dev/null | head -20
```


## Completion Checklist

- [ ] All pages have unique `<title>` tags (≤60 chars) *(if A)*
- [ ] All pages have unique meta descriptions (120–160 chars) *(if A)*
- [ ] Open Graph tags present on all key pages *(if A)*
- [ ] Twitter Card tags present *(if A)*
- [ ] `sitemap.xml` generated and accessible *(if B)*
- [ ] `robots.txt` created with sitemap reference *(if C)*
- [ ] Canonical tags on all pages *(if D)*
- [ ] Structured data validated with Rich Results Test *(if E)*
- [ ] All images have descriptive alt text *(if F)*
- [ ] LCP image has `fetchpriority="high"` and no `loading="lazy"` *(if F/G)*
- [ ] `font-display: swap` set for custom fonts *(if G)*
- [ ] External origins have `<link rel="preconnect">` *(if G)*
- [ ] hreflang tags present on all localized pages including `x-default` *(if H)*
- [ ] Each page has exactly one `<h1>` *(if I)*
- [ ] No orphan pages without inbound internal links *(if J)*
- [ ] noindex on non-indexable pages *(if K)*
- [ ] **Verified: validate at least 2–3 pages with Google's Rich Results Test and a meta tag checker**
