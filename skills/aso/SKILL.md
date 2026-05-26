---
name: aso
description: Make a site agent-ready using Cloudflare AI Search and the isitagentready.com checklist. Audits the current site, then implements: Cloudflare AI Search (hybrid vector + keyword), markdown content negotiation (acceptmarkdown.com), robots.txt AI rules, sitemap, MCP Server Card, and Agent Skills discovery. Use when asked to add search, make a site agent-ready, or optimize for AI agents.
argument-hint: [site URL or project directory, optional]
---

# ASO

You are making a site fully agent-ready using Cloudflare AI Search and the standards defined at isitagentready.com. Work through each phase in order.

**Target:** {{args}}

## Phase 1: Detect Current State

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

## Phase 2: Audit Against isitagentready.com

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

## Phase 3: Cloudflare AI Search Setup

### 3a. Install Wrangler (if not already installed)

```bash
bun add -D wrangler
```

### 3b. Create a search instance

```bash
bunx wrangler ai-search create <project-name>-search
```

For binding config, Worker code, indexing, and hybrid search setup, see [references/search-setup.md](references/search-setup.md)

## Phase 4: Markdown Content Negotiation

AI agents save tokens when they receive Markdown instead of HTML. Implement `Accept: text/markdown` content negotiation per [acceptmarkdown.com](https://acceptmarkdown.com/).

For framework-specific implementation, see [references/markdown-negotiation.md](references/markdown-negotiation.md)

## Phase 5: robots.txt AI Rules

Add AI bot directives alongside a sitemap reference, explicitly allowing or blocking AI crawlers as needed.

See [references/robots-sitemap.md](references/robots-sitemap.md) for the full robots.txt templates

## Phase 6: XML Sitemap

Generate and serve a sitemap at `/sitemap.xml`; reference it in `robots.txt`.

See [references/robots-sitemap.md](references/robots-sitemap.md) for framework-specific sitemap generation

## Phase 7: MCP Server Card

Publish at `/.well-known/mcp.json` so AI agents can discover your MCP capabilities.

See [references/discovery-files.md](references/discovery-files.md) for the mcp.json template

## Phase 8: Agent Skills Listing

Publish at `/.well-known/agent-skills.json` to advertise what actions agents can take on your site.

See [references/discovery-files.md](references/discovery-files.md) for the agent-skills.json template

## Phase 9: Verification

Run verification checks and confirm each passes — see [references/verification.md](references/verification.md)

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
