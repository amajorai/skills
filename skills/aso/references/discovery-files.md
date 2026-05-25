# Discovery File Templates — MCP Server Card & Agent Skills

## Phase 7: mcp.json Template

Publish at `/.well-known/mcp.json`:

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

If you're using `cloudflare/agents-starter`, the MCP endpoint is auto-provisioned:

```typescript
// wrangler.jsonc: AI Search instances include a built-in MCP endpoint
// Access it at: https://<worker>.workers.dev/mcp
```

## Phase 8: agent-skills.json Template

Publish at `/.well-known/agent-skills.json`:

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
