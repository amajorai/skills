---
name: og-images
description: Add dynamic Open Graph images to any web app using an edge function. No third-party service dependency. Generates per-page social preview images for Twitter/X, LinkedIn, Slack, and iMessage link previews. Use when sharing the app's links looks blank or generic.
argument-hint: <pages or routes that need OG images>
---

# OG Images

You are implementing dynamic OG image generation via an edge function. No external service: images are generated at the edge on demand and cached.

**Target pages:** {{args}}


## Phase 1: Audit Current State

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | Existing `<meta>` tags, any current OG image setup, head/layout components |
| 2 | Routing structure, what data is available per page (title, description, author, etc.) |

Identify: which pages need unique images vs. a shared template, what dynamic data to include.


## Phase 2: Design the Template

Ask the user (one batch):

- **Brand**: What colors, fonts, and logo to use?
- **Content per image**: Title only? Title + description? Title + author + date (for blog posts)?
- **Style**: Dark background or light? Minimal or illustrated?
- **Pages**: Which routes get dynamic images vs. a static fallback?

Sketch the layout in ASCII before building:

```
┌─────────────────────────────────┐
│  [Logo]                         │
│                                 │
│  Page Title Here                │
│  Subtitle or description        │
│                                 │
│  yourdomain.com                 │
└─────────────────────────────────┘
```

Confirm with the user before implementing.


## Phase 3: Implement the Image Endpoint

### Framework-specific setup

**Next.js**: use `next/og` with `ImageResponse`:
```
app/og/route.tsx  (or pages/api/og.tsx for Pages Router)
```

**Hono / Bun server**: use `@vercel/og` or `satori` + `sharp`:
```bash
command -v bun >/dev/null 2>&1 && PM=bun || (command -v pnpm >/dev/null 2>&1 && PM=pnpm || PM=npm)
$PM add satori @resvg/resvg-js
```

**Cloudflare Workers**: use `workers-og` (built on Satori, designed for the Workers edge runtime):
```bash
$PM add workers-og
```

### Implementation steps

1. Create the `/og` route that accepts query params: `?title=...&description=...`
2. Build the JSX template using inline styles (Satori only supports a subset of CSS: no flexbox gap, no grid)
3. Embed the font as a base64 string or fetch it from a CDN (fonts must be loaded explicitly)
4. Set cache headers: `Cache-Control: public, max-age=86400, s-maxage=604800`
5. Return the image as `image/png` with 1200×630 dimensions


## Phase 4: Wire Up Meta Tags

For each page:

1. Set `og:image` to the `/og` endpoint URL with page-specific query params
2. Also set `twitter:card: summary_large_image`, `twitter:image`
3. Ensure `og:title`, `og:description`, and `og:url` are present on every page
4. Use absolute URLs (not relative) for all OG tags

For dynamic routes (blog posts, user profiles), populate params from the page's data fetch.


## Phase 5: Verify

Test with real tools, not just by looking at the HTML source:

- [ ] Open [opengraph.xyz](https://www.opengraph.xyz) and enter the URL: confirm image renders
- [ ] Twitter card validator: paste URL, confirm large image appears
- [ ] Check iMessage / Slack preview by sharing a link
- [ ] Verify 1200×630 dimensions (not stretched)
- [ ] Confirm image loads in < 1s on first hit (edge cold start)
- [ ] Confirm subsequent hits are fast (CDN cache hit)


## Completion Report

- Endpoint created at `/og` with accepted params
- Pages wired up with correct meta tags
- Font and brand assets embedded
- Cache headers set
- Verified with external validators
