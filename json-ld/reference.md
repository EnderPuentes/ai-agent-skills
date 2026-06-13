# JSON-LD — Reference & Official Documentation

This file complements the json-ld skill with official documentation links and extra context for indexing.

## W3C JSON-LD (normative)

- **JSON-LD 1.1** (syntax): https://www.w3.org/TR/json-ld11/
- **JSON-LD 1.1 Processing Algorithms and API**: https://www.w3.org/TR/json-ld11-api/
- **JSON-LD 1.1 Framing**: https://www.w3.org/TR/json-ld11-framing/
- **JSON-LD 1.0** (legacy): https://www.w3.org/TR/json-ld/
- **JSON-LD home**: https://json-ld.org/
- **JSON-LD Playground**: https://json-ld.org/playground/
- **JSON-LD Test Suite**: https://json-ld.org/test-suite/
- **Community Group**: https://www.w3.org/community/json-ld/
- **GitHub (spec repos)**: https://github.com/json-ld

## Keywords (JSON-LD 1.1)

Core syntax tokens from the spec (§ 9.16 Keywords):

`@base`, `@container`, `@context`, `@direction`, `@graph`, `@id`, `@import`, `@included`, `@index`, `@json`, `@language`, `@list`, `@nest`, `@none`, `@prefix`, `@propagate`, `@protected`, `@reverse`, `@set`, `@type`, `@value`, `@version`, `@vocab`

Embedding in HTML: use `<script type="application/ld+json">` (§ 7.2 Embedding JSON-LD in HTML Documents). Multiple script elements are merged into one dataset.

Media type: `application/ld+json`

## schema.org vocabulary

- **Home**: https://schema.org/
- **Full hierarchy**: https://schema.org/docs/full.html
- **Getting started**: https://schema.org/docs/gs.html
- **JSON-LD examples**: https://schema.org/docs/schemas.html

Common types for web applications:

| Type | Docs |
|------|------|
| Organization | https://schema.org/Organization |
| WebSite | https://schema.org/WebSite |
| WebPage | https://schema.org/WebPage |
| Article | https://schema.org/Article |
| BlogPosting | https://schema.org/BlogPosting |
| NewsArticle | https://schema.org/NewsArticle |
| BreadcrumbList | https://schema.org/BreadcrumbList |
| Product | https://schema.org/Product |
| FAQPage | https://schema.org/FAQPage |
| LocalBusiness | https://schema.org/LocalBusiness |
| Person | https://schema.org/Person |
| VideoObject | https://schema.org/VideoObject |
| Recipe | https://schema.org/Recipe |
| Event | https://schema.org/Event |
| Review | https://schema.org/Review |
| SearchAction | https://schema.org/SearchAction |

## Google Search Central (structured data)

Google recommends JSON-LD when possible. Use Google docs for **required/recommended** fields per rich result type.

- **Introduction**: https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data
- **General guidelines**: https://developers.google.com/search/docs/appearance/structured-data/sd-policies
- **Feature gallery** (all rich result types): https://developers.google.com/search/docs/appearance/structured-data/search-gallery
- **Rich Results Test**: https://search.google.com/test/rich-results
- **Article**: https://developers.google.com/search/docs/appearance/structured-data/article
- **Breadcrumb**: https://developers.google.com/search/docs/appearance/structured-data/breadcrumb
- **FAQ**: https://developers.google.com/search/docs/appearance/structured-data/faqpage
- **Product**: https://developers.google.com/search/docs/appearance/structured-data/product
- **Recipe**: https://developers.google.com/search/docs/appearance/structured-data/recipe
- **Video**: https://developers.google.com/search/docs/appearance/structured-data/video
- **Organization / logo**: https://developers.google.com/search/docs/appearance/structured-data/logo
- **Sitelinks search box**: https://developers.google.com/search/docs/appearance/structured-data/sitelinks-searchbox

Note: `data-vocabulary.org` markup is no longer eligible for Google rich results.

## Validation and developer tools

| Tool | URL | Use |
|------|-----|-----|
| JSON-LD Playground | https://json-ld.org/playground/ | Expand, compact, frame, visualize graphs |
| Google Rich Results Test | https://search.google.com/test/rich-results | Google eligibility and preview |
| Schema Markup Validator | https://validator.schema.org/ | schema.org property validation |
| Search Console | https://search.google.com/search-console | Production monitoring |

## JavaScript / TypeScript libraries

- **jsonld** (Node.js): https://www.npmjs.com/package/jsonld — expand, compact, flatten, frame, toRDF
- **jsonld.js** (browser + Node): https://github.com/digitalbazaar/jsonld.js
- **TypeScript types**: define interfaces per schema type or use generic `Record<string, unknown>` for script injection

## Framework patterns

### Next.js App Router

- Site-wide `@graph` (Organization + WebSite) in `app/layout.tsx`
- Page-specific types in `app/**/page.tsx` or colocated `JsonLd` component
- Serialize with `JSON.stringify`; never hand-write JSON strings in templates

### React / Vite

- Same `<script type="application/ld+json">` pattern via `dangerouslySetInnerHTML` or SSR head injection (e.g. `react-helmet-async`)

### CMS / SSG

- Generate JSON-LD at build time from frontmatter (title, date, author, image)
- Keep `datePublished` / `dateModified` in ISO 8601 (`YYYY-MM-DD` or full datetime)

## Processing algorithms (API spec)

When building tools (not typical page markup):

- **Expand** — normalize to expanded document form
- **Compact** — shorten using a context
- **Flatten** — single default graph with blank-node labels
- **Frame** — reshape to a template (Framing spec)
- **From RDF** / **To RDF** — RDF dataset interop

API entry: https://www.w3.org/TR/json-ld11-api/

## Related standards

- **JSON** (RFC 8259): https://www.rfc-editor.org/rfc/rfc8259
- **RDF 1.1 Concepts**: https://www.w3.org/TR/rdf11-concepts/
- **Linked Data**: https://www.w3.org/DesignIssues/LinkedData.html
- **YAML-LD** (human authoring): https://json-ld.org/yaml-ld/
- **CBOR-LD** (binary): https://json-ld.org/cbor-ld/

## Related skills

- **pagespeed-insights**: Performance auditing; structured data size is minor but avoid huge inline JSON blocks on critical paths.
- **typescript**: Type-safe schema builders and CMS field mapping.
