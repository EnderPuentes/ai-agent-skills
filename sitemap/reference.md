# Sitemap — Reference & Official Documentation

This file complements the sitemap skill with official documentation links and extra context for indexing.

## Official specification

- **Home**: https://www.sitemaps.org/
- **Protocol (XML format)**: https://www.sitemaps.org/protocol.html
- **FAQ**: https://www.sitemaps.org/faq.php
- **Terms**: https://www.sitemaps.org/terms.php
- **Version**: Sitemap 0.90 (Attribution-ShareAlike CC; supported by Google, Bing, Yahoo historically)

## XML schemas

- **urlset**: http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd
- **sitemapindex**: http://www.sitemaps.org/schemas/sitemap/0.9/siteindex.xsd

## Required tags (urlset)

Namespace: `xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"`

| Element | Parent | Required |
|---------|--------|----------|
| `urlset` | — | root |
| `url` | urlset | per entry |
| `loc` | url | yes |

Optional per URL: `lastmod`, `changefreq`, `priority`.

## changefreq values

`always`, `hourly`, `daily`, `weekly`, `monthly`, `yearly`, `never` — hints only, not crawl commands.

## Size limits (FAQ)

| Limit | Value |
|-------|-------|
| URLs per sitemap | 50,000 |
| Uncompressed size | 50 MB (52,428,800 bytes) |
| Sitemaps per index | 50,000 |
| Index uncompressed size | 50 MB |

Compress with **gzip** for transfer; uncompressed size still applies.

## lastmod (FAQ)

- W3C Datetime: `2004-09-22T14:12:14+00:00` or date-only `2004-09-22`.
- Static files: actual file mtime.
- Dynamic pages: when underlying data changed (or best approximation).
- Include time portion if site changes frequently.

## Entity escaping

| Char | Escape |
|------|--------|
| `&` | `&amp;` |
| `'` | `&apos;` |
| `"` | `&quot;` |
| `<` | `&lt;` |
| `>` | `&gt;` |

URLs must also conform to RFC-3986 (URI) / RFC-3987 (IRI). Encoding: **UTF-8**.

## Alternative formats (protocol)

- **RSS 2.0** / **Atom 0.3 / 1.0** — syndication feeds as sitemaps (recent URLs only).
- **Plain text** — one URL per line, UTF-8, fully qualified, max 50k lines / 50 MB.

## Placement & scope (FAQ)

- Strongly recommended: `https://example.com/sitemap.xml` (root).
- All URLs in a sitemap must be on the **same host** as the sitemap file.
- Subpath sitemaps only include URLs under that path prefix.
- Cross-subdomain: use Search Console "cross submit" or separate properties per host.

## Submission (FAQ)

After creating sitemap:

1. Submit to search engines (Search Console, Bing, ping).
2. Add to `robots.txt`: `Sitemap: https://example.com/sitemap.xml`

Does **not** guarantee inclusion in search results.

## FAQ highlights

- **priority** does not change ranking vs other websites — only relative within site.
- **URL order** in file does not affect crawling.
- **http vs https**: list only one canonical version.
- **Session IDs**: remove from URLs.
- **Frames**: include both frameset and frame content URLs.

## Google Search Central

- Sitemaps overview: https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap
- Search Console sitemap report: submit and monitor errors
- Google supports sitemap index, gzip, `lastmod` (treated as hint)

## Next.js

- **MetadataRoute.Sitemap**: `app/sitemap.ts` → `/sitemap.xml`
- Fields: `url`, `lastModified`, `changeFrequency`, `priority`
- **generateSitemaps** + dynamic `sitemap.ts` for multiple sitemap files (large sites)
- **robots.ts**: `app/robots.ts` can reference sitemap URL

## Comparison stack

| File | Standard | Audience |
|------|----------|----------|
| robots.txt | de facto / RFC 9309 | Crawl permission |
| sitemap.xml | sitemaps.org 0.9 | Search engine URL discovery |
| JSON-LD | W3C + schema.org | Page semantics (json-ld) |
| llms.txt | llmstxt.org community | AI agent curated index (llms-txt) |

## Related skills

- **json-ld** — structured data on indexed pages.
- **llms-txt** — llms.txt for answer engines.
