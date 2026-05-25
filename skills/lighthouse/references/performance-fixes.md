# Performance Fixes

## Images

- Add explicit `width` and `height` to all `<img>` tags to prevent CLS
- Add `loading="lazy"` to below-the-fold images
- Add `fetchpriority="high"` to the LCP image
- Convert large PNGs/JPGs to WebP/AVIF using `<picture>` with fallback
- Use `srcset` for responsive images
- Compress images if oversized (use `sharp` CLI or equivalent)

## JavaScript

- Move non-critical `<script>` tags to `defer` or `async`
- Identify and remove unused JS: check Lighthouse "Reduce unused JavaScript" audit
- Split large bundles: check if the framework supports dynamic `import()`
- Add `rel="modulepreload"` for critical JS modules

## CSS

- Move `<link rel="stylesheet">` for non-critical CSS to load asynchronously:
  ```html
  <link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
  ```
- Remove unused CSS if the audit flags it
- Inline critical (above-the-fold) CSS in `<head>` for the fastest FCP

## Fonts

- Add `rel="preconnect"` for Google Fonts or CDN font hosts
- Add `font-display: swap` to all `@font-face` declarations
- Preload the primary font file:
  ```html
  <link rel="preload" href="/fonts/main.woff2" as="font" type="font/woff2" crossorigin>
  ```

## Caching

- Add `Cache-Control` headers for static assets (images, JS, CSS): `max-age=31536000, immutable`
- If using a framework with content hashing in filenames, enable long-lived caching
- Add `ETag` or `Last-Modified` for HTML responses

## Server / TTFB

- Check if responses are gzip/brotli compressed: add if missing
- Add `rel="dns-prefetch"` or `rel="preconnect"` for third-party origins
- If server-side: check for slow database queries or missing indexes
- Consider adding a CDN or edge caching layer if TTFB is consistently > 800ms

## Third-party scripts

- Load analytics, chat widgets, and ad scripts with `defer` or move to `async`
- Use Partytown or similar to offload heavy third-party scripts to a web worker
