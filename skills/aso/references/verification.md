# Verification Checks & Completion Checklist

## AI Search

```bash
# Test search endpoint
curl -X POST https://yourworker.workers.dev/search \
  -H "Content-Type: application/json" \
  -d '{"query": "getting started"}'
# Should return results array
```

## Markdown Negotiation

```bash
curl -sI -H "Accept: text/markdown" https://yoursite.com/
# Expect: Content-Type: text/markdown; charset=utf-8
# Expect: Vary: Accept
```

## robots.txt

```bash
curl -s https://yoursite.com/robots.txt | grep -E "(GPTBot|ClaudeBot|Sitemap)"
```

## Sitemap

```bash
curl -sI https://yoursite.com/sitemap.xml
# Expect: 200 OK, Content-Type: application/xml
```

## MCP Discovery

```bash
curl -s https://yoursite.com/.well-known/mcp.json | jq .
```

## Agent Skills

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
