# Meta Tag Templates

## Next.js App Router

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

## Next.js Pages Router

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

## Astro

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

## Plain HTML / other frameworks

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
