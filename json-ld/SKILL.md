---
name: json-ld
description: Add and validate JSON-LD structured data with schema.org for SEO rich results. Use when implementing structured data, schema markup, application/ld+json, Organization, WebSite, Article, Product, FAQ, BreadcrumbList, or when the user mentions JSON-LD, schema.org, rich snippets, or Google structured data.
---

# JSON-LD

## Overview

JSON-LD (JSON for Linked Data) is a W3C standard for embedding machine-readable metadata in JSON. On the web it is most often used with the **schema.org** vocabulary inside `<script type="application/ld+json">` so search engines understand page content and may show **rich results**.

**Principles**: Valid JSON first; `@context` maps terms to IRIs; `@type` declares entity kind; prefer absolute URLs for `@id` and links; markup must match visible page content.

Reference: [W3C JSON-LD 1.1](https://www.w3.org/TR/json-ld11/) · [schema.org](https://schema.org/) · [Google structured data intro](https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data)

---

## Minimal document

Every JSON-LD document needs `@context` and usually `@type`:

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Acme Corp",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png"
}
```

For schema.org, use `"@context": "https://schema.org"` (or `"https://schema.org/"`).

---

## Embedding in HTML

Per the JSON-LD spec, place JSON in a script element with `type="application/ld+json"`. Multiple scripts on one page are merged into a single dataset.

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebPage",
  "name": "About Us",
  "url": "https://example.com/about"
}
</script>
```

- Put in `<head>` or `<body>`; both are valid.
- Content must be **valid JSON** (double quotes, no trailing commas, no comments).
- Escape `</script>` inside strings if needed (e.g. split as `<\/script>`).

### Next.js (App Router)

Use a dedicated component or inline in `layout.tsx` / `page.tsx`:

```tsx
export function JsonLd({ data }: { data: Record<string, unknown> }) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(data) }}
    />
  )
}
```

- Build the object in TypeScript; `JSON.stringify` handles escaping.
- For dynamic pages, derive `name`, `url`, `datePublished`, etc. from route params and CMS data.
- Site-wide schemas (Organization, WebSite) belong in root `layout.tsx`; page-specific schemas in each `page.tsx`.

---

## Core keywords (JSON-LD 1.1)

| Keyword | Purpose |
|---------|---------|
| `@context` | Vocabulary and term mappings (required at document or node level) |
| `@type` | Entity type (e.g. `WebPage`, `Person`, `Product`) |
| `@id` | Canonical IRI for the entity; use for cross-references |
| `@graph` | Array of node objects in one document |
| `@language` / `@value` | Language-tagged or typed literals (advanced) |

**Node references** — link entities without duplication:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "WebSite",
      "@id": "https://example.com/#website",
      "url": "https://example.com",
      "name": "Example",
      "publisher": { "@id": "https://example.com/#organization" }
    },
    {
      "@type": "Organization",
      "@id": "https://example.com/#organization",
      "name": "Example Inc.",
      "url": "https://example.com"
    }
  ]
}
```

Use stable fragment IDs (`#website`, `#organization`) for site-level entities.

---

## Common schema.org types (web)

Add only types that match **visible** page content. For Google rich results, follow [Google's feature docs](https://developers.google.com/search/docs/appearance/structured-data/search-gallery) (required/recommended fields differ per type).

| Type | Typical use |
|------|-------------|
| **Organization** | Company / brand (logo, sameAs social URLs) |
| **WebSite** | Site identity; optional `SearchAction` for sitelinks search box |
| **WebPage** | Generic page; pair with `isPartOf` → WebSite |
| **Article** / **NewsArticle** / **BlogPosting** | Articles and blog posts |
| **BreadcrumbList** | Breadcrumb navigation |
| **Product** | E-commerce product pages |
| **FAQPage** | FAQ sections with `Question` / `Answer` |
| **LocalBusiness** | Physical business (address, hours, geo) |
| **Person** | Author or profile pages |
| **VideoObject** | Video embed pages |
| **Recipe** | Recipe pages |

### WebSite + Organization (root layout)

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "WebSite",
      "@id": "https://example.com/#website",
      "url": "https://example.com",
      "name": "Example",
      "publisher": { "@id": "https://example.com/#organization" }
    },
    {
      "@type": "Organization",
      "@id": "https://example.com/#organization",
      "name": "Example Inc.",
      "url": "https://example.com",
      "logo": {
        "@type": "ImageObject",
        "url": "https://example.com/logo.png"
      }
    }
  ]
}
```

### Article / BlogPosting

```json
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "headline": "Post title",
  "description": "Short summary",
  "image": ["https://example.com/og.jpg"],
  "datePublished": "2025-06-13",
  "dateModified": "2025-06-13",
  "author": {
    "@type": "Person",
    "name": "Jane Doe",
    "url": "https://example.com/about"
  },
  "publisher": { "@id": "https://example.com/#organization" },
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://example.com/blog/post-slug"
  }
}
```

### BreadcrumbList

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "https://example.com"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Blog",
      "item": "https://example.com/blog"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "Post title"
    }
  ]
}
```

---

## Validation workflow

1. **Syntax** — ensure valid JSON (no comments, trailing commas).
2. **[JSON-LD Playground](https://json-ld.org/playground/)** — expand/compact; debug `@context` and graph structure.
3. **[Google Rich Results Test](https://search.google.com/test/rich-results)** — eligibility for Google rich results (when targeting SEO).
4. **Search Console** — monitor rich result status after deploy.

For programmatic checks in Node.js, use the [`jsonld`](https://www.npmjs.com/package/jsonld) package (`jsonld.expand`, `jsonld.compact`).

---

## Google / SEO guidelines (summary)

- **JSON-LD is Google's recommended format** for structured data when you can choose.
- Markup must describe content **visible on the page**; no hidden or misleading data.
- Prefer complete required fields over stuffing optional properties with weak data.
- Use [Google's type-specific docs](https://developers.google.com/search/docs/appearance/structured-data/search-gallery) for required properties — they can be stricter than schema.org.
- `data-vocabulary.org` is **deprecated** for Google rich results; use schema.org.

---

## Common mistakes

- **Invalid JSON** — comments, single quotes, trailing commas break parsers.
- **Relative URLs** — use absolute URLs for `url`, `image`, `@id`, and `item` in breadcrumbs.
- **Mismatch with UI** — `headline`, prices, ratings, or FAQ answers that don't match the page.
- **Duplicate conflicting graphs** — multiple scripts defining the same `@id` with different data.
- **Wrong `@type`** — e.g. `Article` on a product page; pick the type that matches primary content.
- **Missing `@context`** — every standalone document needs it.
- **Escaping in HTML** — raw `</script>` in JSON strings breaks the script tag; use `JSON.stringify`.

---

## Advanced (when needed)

- **Framing** — reshape expanded data with a [JSON-LD frame](https://www.w3.org/TR/json-ld11-framing/) (W3C Framing spec).
- **Custom vocabulary** — define `@context` with term mappings for non–schema.org IRIs.
- **HTTP headers** — `Link` with `rel="alternate"; type="application/ld+json"` for non-HTML resources (API, PDF).
- **RDF interop** — JSON-LD serializes RDF; use expanded form and N-Quads when integrating with RDF tools.

## Related skills

- **sitemap** — XML sitemap for crawl discovery.
- **llms-txt** — llms.txt for AI/agent context (complements, does not replace, JSON-LD).

---

## Additional resources

- [reference.md](reference.md) — W3C specs, schema.org, Google Search Central, tools, npm libraries.
- W3C JSON-LD 1.1: https://www.w3.org/TR/json-ld11/
- JSON-LD Playground: https://json-ld.org/playground/
- schema.org: https://schema.org/
- Google structured data: https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data
