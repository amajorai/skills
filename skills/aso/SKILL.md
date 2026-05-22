---
name: aso
description: Make a site agent-ready using Cloudflare AI Search and the isitagentready.com checklist. Audits the current site, then implements: Cloudflare AI Search (hybrid vector + keyword), markdown content negotiation (acceptmarkdown.com), robots.txt AI rules, sitemap, MCP Server Card, and Agent Skills discovery. Use when asked to add search, make a site agent-ready, or optimize for AI agents.
argument-hint: [site URL or project directory, optional]
---

# aso — Agent-Ready Search Optimization

You are making a site fully agent-ready using Cloudflare AI Search and the standards defined at isitagentready.com. Work through each phase in order.

**Target:** {{args}}


## Phase 0: Detect Current State

Run silently before asking anything. Check what's already in place:

```bash
# Detect framework / runtime
ls package.json wrangler.jsonc wrangler.toml next.config.* astro.config.* 2>/dev/null
cat package.json 2>/dev/null | grep -E '"(next|astro|remix|nuxt|svelte|hono|workers-sdk|wrangler)"'

# Check for existing Cloudflare config
cat wrangler.jsonc 2>/dev/null || cat wrangler.toml 2>/dev/null

# Check robots.txt
cat public/robots.txt 2>/dev/null || cat static/robots.txt 2>/dev/null

# Check for sitemap
ls public/sitemap*.xml static/sitemap*.xml 2>/dev/null

# Check for MCP / agent discovery files
ls public/.well-known/ 2>/dev/null
cat public/.well-known/mcp.json 2>/dev/null
cat public/.well-known/ai-plugin.json 2>/dev/null

# Check for existing AI Search binding
grep -r "ai_search" wrangler.jsonc wrangler.toml 2>/dev/null

# Check for markdown negotiation
grep -r "text/markdown" src/ app/ pages/ 2>/dev/null | head -10
```

Note: Is this a Workers project? What framework? What's already done vs. missing?


## Phase 1: Audit Against isitagentready.com

Check each category and mark as DONE / MISSING:

**Discoverability**
- robots.txt with AI bot rules and Sitemap reference
- XML sitemap
- Link response headers (`Link: <URL>; rel="alternate"; type="text/markdown"`)

**Content Accessibility**
- Markdown content negotiation (`Accept: text/markdown` → respond with `.md`)

**Bot Access Control**
- AI bot rules in robots.txt (GPTBot, ClaudeBot, PerplexityBot, etc.)
- Web Bot Auth (if needed for authenticated content)

**Protocol Discovery**
- MCP Server Card at `/.well-known/mcp.json`
- Agent Skills listing
- API Catalog (if the site has an API)

**Search**
- Cloudflare AI Search instance with hybrid indexing

Show the user a scored summary before proceeding:
```
Discoverability:   [2/3 done]
Content:           [0/1 done]
Bot access:        [1/2 done]
Protocol:          [0/2 done]
Search:            [0/1 done]
```

Ask: "Which of these do you want me to implement? I can do all of them, or we can start with the highest-impact ones first." Default recommendation: all.


## Phase 2: Cloudflare AI Search Setup

### 2a. Install Wrangler (if not already installed)

```bash
bun add -D wrangler
```

### 2b. Create a search instance

```bash
npx wrangler ai-search create <project-name>-search
```

### 2c. Add the binding to `wrangler.jsonc`

```jsonc
{
  "ai_search_namespaces": [
    {
      "binding": "AI_SEARCH",
      "namespace": "<project-name>-search"
    }
  ]
}
```

For multi-tenant or per-agent use, also add a namespace binding for dynamic creation:

```jsonc
{
  "ai_search_namespaces": [
    {
      "binding": "SEARCH_NS",
      "namespace": "<project-name>"
    }
  ]
}
```

### 2d. Create the search Worker (`src/search.ts`)

```typescript
export interface Env {
  AI_SEARCH: any;
}

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);

    if (request.method === "POST" && url.pathname === "/search") {
      const { query, filters } = await request.json<{ query: string; filters?: Record<string, string> }>();

      const results = await env.AI_SEARCH.search({
        query,
        ai_search_options: {
          boost_by: [{ field: "timestamp", direction: "desc" }],
          ...(filters && { metadata_filter: filters }),
        },
      });

      return Response.json(results);
    }

    return new Response("Not found", { status: 404 });
  },
};
```

### 2e. Index content

Upload documents with metadata for filtering and boosting:

```typescript
const instance = env.AI_SEARCH;

// Upload a document
await instance.items.uploadAndPoll("doc-id", markdownContent, {
  metadata: {
    category: "docs",
    version: "v2",
    timestamp: Date.now().toString(),
  },
});
```

### 2f. Enable hybrid search (recommended for best results)

Create the instance with both keyword and vector search:

```typescript
await env.SEARCH_NS.create({
  id: "my-instance",
  index_method: { keyword: true, vector: true },
  indexing_options: { keyword_tokenizer: "porter" },
  retrieval_options: { keyword_match_mode: "or" },
  fusion_method: "rrf",
  reranking: true,
  reranking_model: "@cf/baai/bge-reranker-base",
});
```

**Tokenizer guide:**
- `porter` — natural language prose (stems words: "running" → "run")
- `trigram` — code search (character-level substring matching)

**Fusion methods:**
- `rrf` (reciprocal rank fusion) — balanced hybrid, best default
- `max` — prefer whichever signal scores higher


## Phase 3: Markdown Content Negotiation

AI agents save tokens when they receive Markdown instead of HTML. Implement `Accept: text/markdown` content negotiation per [acceptmarkdown.com](https://acceptmarkdown.com/).

### For a Cloudflare Worker / Hono

```typescript
app.get("*", async (c) => {
  const accept = c.req.header("Accept") ?? "";

  if (accept.includes("text/markdown")) {
    const md = await fetchMarkdownVersion(c.req.url);
    return c.text(md, 200, {
      "Content-Type": "text/markdown; charset=utf-8",
      "Vary": "Accept",
    });
  }

  return c.html(await fetchHtmlVersion(c.req.url));
});
```

### For Next.js (middleware)

```typescript
// middleware.ts
import { NextRequest, NextResponse } from "next/server";

export function middleware(request: NextRequest) {
  const accept = request.headers.get("accept") ?? "";
  if (accept.includes("text/markdown")) {
    const mdUrl = new URL(request.url);
    mdUrl.pathname = mdUrl.pathname.replace(/\/?$/, ".md");
    return NextResponse.rewrite(mdUrl);
  }
}
```

Add `Link` headers to HTML responses so agents can discover the Markdown variant:

```typescript
headers: {
  "Link": `<${markdownUrl}>; rel="alternate"; type="text/markdown"`,
  "Vary": "Accept",
}
```

**Test with:**
```bash
curl -sI -H "Accept: text/markdown" https://yoursite.com/docs/getting-started
# Should return Content-Type: text/markdown; charset=utf-8
```


## Phase 4: robots.txt AI Rules

Add AI bot directives alongside a sitemap reference:

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

If you want to block AI training crawlers but allow AI assistants:

```
# Block training crawlers
User-agent: CCBot
Disallow: /

# Allow assistant-mode crawlers explicitly
User-agent: GPTBot
Allow: /docs/
```


## Phase 5: XML Sitemap

### For static sites (Astro, Hugo, etc.)

Most frameworks auto-generate a sitemap via plugin — enable it:

```js
// astro.config.mjs
import sitemap from "@astrojs/sitemap";
export default defineConfig({
  site: "https://yoursite.com",
  integrations: [sitemap()],
});
```

### For Cloudflare Workers / manual

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


## Phase 6: MCP Server Card

Publish at `/.well-known/mcp.json` so AI agents can discover your MCP capabilities:

```json
{
  "name": "Your Site Name",
  "description": "What your site/API does in one sentence.",
  "version": "1.0.0",
  "mcp_endpoint": "https://yoursite.com/mcp",
  "capabilities": {
    "tools": true,
    "resources": true,
    "prompts": false
  },
  "authentication": {
    "type": "none"
  },
  "contact": {
    "url": "https://yoursite.com/contact"
  }
}
```

If you're using `cloudflare/agents-starter`, the MCP endpoint is auto-provisioned. Expose it:

```typescript
// wrangler.jsonc — AI Search instances include a built-in MCP endpoint
// Access it at: https://<worker>.workers.dev/mcp
```


## Phase 7: Agent Skills Listing

Publish at `/.well-known/agent-skills.json` to advertise what actions agents can take on your site:

```json
{
  "skills": [
    {
      "name": "search",
      "description": "Search the knowledge base with natural language",
      "endpoint": "https://yoursite.com/search",
      "method": "POST",
      "input_schema": {
        "type": "object",
        "properties": {
          "query": { "type": "string" },
          "filters": { "type": "object" }
        },
        "required": ["query"]
      }
    }
  ]
}
```


## Phase 8: Verification

Run these checks and confirm each passes:

**Cloudflare AI Search**
```bash
# Test search endpoint
curl -X POST https://yourworker.workers.dev/search \
  -H "Content-Type: application/json" \
  -d '{"query": "getting started"}'
# Should return results array
```

**Markdown negotiation**
```bash
curl -sI -H "Accept: text/markdown" https://yoursite.com/
# Expect: Content-Type: text/markdown; charset=utf-8
# Expect: Vary: Accept
```

**robots.txt**
```bash
curl -s https://yoursite.com/robots.txt | grep -E "(GPTBot|ClaudeBot|Sitemap)"
```

**Sitemap**
```bash
curl -sI https://yoursite.com/sitemap.xml
# Expect: 200 OK, Content-Type: application/xml
```

**MCP discovery**
```bash
curl -s https://yoursite.com/.well-known/mcp.json | jq .
```

**Agent Skills**
```bash
curl -s https://yoursite.com/.well-known/agent-skills.json | jq .
```

**Full audit:** Paste your URL at https://isitagentready.com/ and confirm all checks pass.


## Completion Checklist

- [ ] Cloudflare AI Search instance created and bound
- [ ] Hybrid search enabled (keyword + vector, RRF fusion)
- [ ] Content indexed with metadata
- [ ] Search endpoint tested and returning results
- [ ] `Accept: text/markdown` content negotiation implemented
- [ ] `Link` header pointing to Markdown variant added
- [ ] robots.txt has AI bot rules and Sitemap reference
- [ ] XML sitemap accessible at `/sitemap.xml`
- [ ] MCP Server Card at `/.well-known/mcp.json`
- [ ] Agent Skills at `/.well-known/agent-skills.json`
- [ ] isitagentready.com audit passes all selected checks
