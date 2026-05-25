# Final Verification & Completion Checklist

## Verification commands

Replace `https://example.com` with the real deployed domain. The `curl` checks only work against a live/deployed URL — if the site is not deployed yet, skip them and rely on local `grep` checks.

```bash
# Sitemap accessible
curl -s https://example.com/sitemap.xml | head -20

# robots.txt accessible
curl -s https://example.com/robots.txt

# Meta tags present on homepage
curl -s https://example.com | grep -E "<title|og:title|description" | head -10

# Images without alt (re-scan)
grep -rn "<img " src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" 2>/dev/null | grep -v "alt=" | head -10

# Pages with multiple h1s
grep -rn "<h1" src/ app/ pages/ --include="*.tsx" --include="*.jsx" --include="*.astro" 2>/dev/null | head -20
```

## Completion checklist

- [ ] All pages have unique `<title>` tags (≤60 chars) *(if A)*
- [ ] All pages have unique meta descriptions (120–160 chars) *(if A)*
- [ ] Open Graph tags present on all key pages *(if A)*
- [ ] Twitter Card tags present *(if A)*
- [ ] `sitemap.xml` generated and accessible *(if B)*
- [ ] `robots.txt` created with sitemap reference *(if C)*
- [ ] Canonical tags on all pages *(if D)*
- [ ] Structured data validated with Rich Results Test *(if E)*
- [ ] All images have descriptive alt text *(if F)*
- [ ] LCP image has `fetchpriority="high"` and no `loading="lazy"` *(if F/G)*
- [ ] `font-display: swap` set for custom fonts *(if G)*
- [ ] External origins have `<link rel="preconnect">` *(if G)*
- [ ] hreflang tags present on all localized pages including `x-default` *(if H)*
- [ ] Each page has exactly one `<h1>` *(if I)*
- [ ] No orphan pages without inbound internal links *(if J)*
- [ ] noindex on non-indexable pages *(if K)*
- [ ] **Verified: validate at least 2–3 pages with Google's Rich Results Test and a meta tag checker**
