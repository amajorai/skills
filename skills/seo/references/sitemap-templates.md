# Sitemap Templates

## Next.js App Router — `app/sitemap.ts`

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

## Next.js Pages Router — `pages/sitemap.xml.tsx`

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

## Astro — `src/pages/sitemap.xml.ts`

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

## Static sites / generic — `public/sitemap.xml`

```xml
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
