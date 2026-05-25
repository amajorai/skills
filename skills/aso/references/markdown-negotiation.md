# Markdown Content Negotiation — Framework Implementations

## Cloudflare Worker / Hono

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

## Next.js (`middleware.ts`)

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

## Link Headers

Add `Link` headers to HTML responses so agents can discover the Markdown variant:

```typescript
headers: {
  "Link": `<${markdownUrl}>; rel="alternate"; type="text/markdown"`,
  "Vary": "Accept",
}
```

## Test

```bash
curl -sI -H "Accept: text/markdown" https://yoursite.com/docs/getting-started
# Should return Content-Type: text/markdown; charset=utf-8
```
