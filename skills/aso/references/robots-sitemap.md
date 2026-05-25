# robots.txt Templates & Sitemap Generation

## Phase 5: robots.txt Templates

### Allow all AI bots

```
# public/robots.txt

User-agent: *
Allow: /

# Explicit AI agent access
User-agent: GPTBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Googlebot-Extended
Allow: /

User-agent: CCBot
Allow: /

# Sitemap
Sitemap: https://yoursite.com/sitemap.xml
```

### Selective block (block training crawlers, allow assistants)

```
# Block training crawlers
User-agent: CCBot
Disallow: /

# Allow assistant-mode crawlers explicitly
User-agent: GPTBot
Allow: /docs/
```

## Phase 6: Sitemap Generation

### Astro (and other static site frameworks)

Most frameworks auto-generate a sitemap via plugin — enable it:

```js
// astro.config.mjs
import sitemap from "@astrojs/sitemap";
export default defineConfig({
  site: "https://yoursite.com",
  integrations: [sitemap()],
});
```

### Cloudflare Workers / manual

```typescript
// src/sitemap.ts
export function generateSitemap(pages: string[], baseUrl: string): string {
  const urls = pages.map((p) => `
  <url>
    <loc>${baseUrl}${p}</loc>
    <changefreq>weekly</changefreq>
  </url>`).join("");

  return `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${urls}
</urlset>`;
}
```

Serve at `/sitemap.xml` and reference it in `robots.txt`.
