# Cloudflare AI Search — Binding Config, Worker Code, Indexing & Hybrid Search

## 3c. Wrangler Binding Config

Single instance binding in `wrangler.jsonc`:

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

## 3d. Search Worker (`src/search.ts`)

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

## 3e. Indexing Content (`uploadAndPoll`)

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

## 3f. Hybrid Search Creation

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
- `porter` - natural language prose (stems words like "running" → "run")
- `trigram` - code search (character-level substring matching)

**Fusion methods:**
- `rrf` (reciprocal rank fusion) - balanced hybrid, best default
- `max` - prefer whichever signal scores higher
