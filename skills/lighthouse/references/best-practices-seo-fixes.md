# Best Practices & SEO Fixes

## Best Practices

### Security

- Ensure the site is served over HTTPS
- Add `rel="noopener noreferrer"` to `target="_blank"` links
- Add a Content Security Policy header (start with `report-only` mode)
- Add `X-Content-Type-Options: nosniff` and `X-Frame-Options: SAMEORIGIN` headers

### Console errors

- Open the Lighthouse JSON and find any logged console errors
- Fix the underlying JS errors: never suppress them silently

### Deprecated APIs

- Update any APIs flagged as deprecated (check the audit's `items` for specifics)

### Doctype

- Ensure `<!DOCTYPE html>` is the first line of every HTML page

## SEO

### Meta tags

- Add `<meta name="description" content="...">` if missing (150–160 chars)
- Add `<title>` if missing or generic
- Add Open Graph tags for social sharing:
  ```html
  <meta property="og:title" content="...">
  <meta property="og:description" content="...">
  <meta property="og:image" content="...">
  <meta property="og:url" content="...">
  ```

### Crawlability

- Ensure `<meta name="robots" content="noindex">` is NOT present on public pages
- Add a `robots.txt` file if missing:
  ```
  User-agent: *
  Allow: /
  Sitemap: https://example.com/sitemap.xml
  ```
- Add an XML sitemap and link it from `robots.txt`
- Ensure links use descriptive text (no "click here" or "read more")

### Structured data

- Add JSON-LD schema markup for the page type (Article, Product, Organization, etc.)
- Validate with Google's Rich Results Test

### Mobile

- Ensure `<meta name="viewport" content="width=device-width, initial-scale=1">` is present
- Confirm tap targets are at least 48×48px with 8px spacing
