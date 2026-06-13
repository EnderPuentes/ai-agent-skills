---
name: sitemap
description: Create and maintain XML sitemaps per sitemaps.org for search engine crawl discovery. Use when adding sitemap.xml, sitemap index, Next.js sitemap.ts, lastmod, changefreq, robots.txt Sitemap directive, or when the user mentions sitemap, URL discovery, or crawl indexing.
---

# SEO — Sitemap

## Overview

A **Sitemap** is an XML file (or RSS/Atom/text alternative) that lists URLs on your site so search engines can discover and crawl them more efficiently. It supplements link-based discovery; it does **not** guarantee indexing.

**Principles**: UTF-8 encoding; absolute URLs with protocol; one host per sitemap; accurate `lastmod`; stay under size limits; place at site root when possible.

Reference: [sitemaps.org protocol](https://www.sitemaps.org/protocol.html) · [FAQ](https://www.sitemaps.org/faq.php)

---

## When to use a sitemap

| Scenario | Approach |
|----------|----------|
| Marketing / docs site | Root `sitemap.xml` with static + dynamic URLs |
| Large site (>50k URLs) | Multiple sitemaps + **sitemap index** |
| CMS with many pages | Generate from DB/CMS at build or request time |
| Recent changes only | Split volatile URLs; use `lastmod` in index for incremental fetch |

Pair with **robots.txt** (`Sitemap: https://example.com/sitemap.xml`) and **Search Console** submission.

---

## XML format (protocol 0.9)

Required structure:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://www.example.com/</loc>
    <lastmod>2025-06-13</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

| Tag | Required | Notes |
|-----|----------|-------|
| `<loc>` | Yes | Full URL with `https://`; max 2,048 chars |
| `<lastmod>` | No | W3C datetime (`YYYY-MM-DD` or `2004-12-23T18:00:15+00:00`); **page** last modified, not sitemap generation time |
| `<changefreq>` | No | Hint only: `always`, `hourly`, `daily`, `weekly`, `monthly`, `yearly`, `never` |
| `<priority>` | No | 0.0–1.0 relative **within your site**; default 0.5; does not affect ranking vs other sites |

**Entity escaping** in XML: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`, `"` → `&quot;`, `'` → `&apos;`.

---

## Limits (FAQ)

- Max **50,000 URLs** per sitemap file.
- Max **50 MB** uncompressed (52,428,800 bytes).
- Use **gzip** compression when serving (file must still be ≤50 MB uncompressed).
- Exceed limits → split into multiple sitemaps + **sitemap index**.

### Sitemap index

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>https://www.example.com/sitemap-posts.xml</loc>
    <lastmod>2025-06-13T12:00:00+00:00</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://www.example.com/sitemap-pages.xml</loc>
    <lastmod>2025-06-01</lastmod>
  </sitemap>
</sitemapindex>
```

Index: max 50,000 sitemaps, max 50 MB. Index `lastmod` = when that sitemap **file** changed (enables incremental crawl).

---

## File location rules

- **Recommended**: `https://example.com/sitemap.xml` (root).
- All `<loc>` URLs must share the **same protocol and host** as the sitemap (e.g. no `http` URLs in an `https` sitemap).
- Subpath sitemap (`/catalog/sitemap.xml`) may only list URLs under `/catalog/`.
- List **one canonical version** of each URL (not both `http` and `https`).
- **No session IDs** in URLs.
- Include **frameset and frame content** URLs if site uses frames (legacy).

---

## Next.js (App Router)

Use `app/sitemap.ts` exporting default async function returning `MetadataRoute.Sitemap`:

```ts
import type { MetadataRoute } from 'next'

const SITE = 'https://example.com'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const staticPages: MetadataRoute.Sitemap = [
    { url: `${SITE}/`, lastModified: new Date(), priority: 1 },
    { url: `${SITE}/about`, lastModified: new Date(), priority: 0.8 },
  ]

  const posts = await fetchPostSlugs()
  const postPages = posts.map((slug) => ({
    url: `${SITE}/blog/${slug}`,
    lastModified: new Date(),
    priority: 0.7,
  }))

  return [...staticPages, ...postPages]
}
```

- Next.js serves at `/sitemap.xml` automatically.
- Use real `lastModified` from CMS when available.
- For multiple sitemaps, use `generateSitemaps` (Next.js 13.3+) or route handlers.

### robots.txt

```
User-agent: *
Allow: /

Sitemap: https://example.com/sitemap.xml
```

---

## After publishing

1. Validate XML against [sitemap.xsd](http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd).
2. Submit in **Google Search Console** / Bing Webmaster Tools.
3. Add `Sitemap:` line to `robots.txt`.
4. Monitor crawl stats; update `lastmod` when content actually changes.

---

## vs JSON-LD and llms.txt

| Artifact | Role |
|----------|------|
| **sitemap.xml** | Enumerate indexable URLs for search crawlers |
| **JSON-LD** (`json-ld`) | Structured semantics per page (rich results) |
| **llms.txt** (`llms-txt`) | Curated map for AI agents (not exhaustive URL list) |

Use all three (`sitemap`, `json-ld`, `llms-txt`) for a complete discoverability stack.

---

## Common mistakes

- **Wrong lastmod** — sitemap generation date instead of page update date.
- **Relative URLs** — must include `https://`.
- **Mixed hosts** — subdomain URLs in www sitemap.
- **Session IDs** in `<loc>`.
- **Duplicate http/https** entries for same page.
- **Priority 1.0 everywhere** — dilutes relative signaling.
- **Unescaped `&`** in query strings — breaks XML.
- **Oversized file** — split before hitting 50k / 50MB.

---

## Related skills

- **json-ld** — structured data on pages listed in the sitemap.
- **llms-txt** — llms.txt for AI assistants (curated, not a sitemap replacement).

---

## Additional resources

- [reference.md](reference.md) — Protocol, FAQ, schemas, Search Console, Next.js.
- Protocol: https://www.sitemaps.org/protocol.html
- FAQ: https://www.sitemaps.org/faq.php
