# llms.txt — Reference & Official Documentation

This file complements the llms-txt skill with official documentation links and extra context for indexing.

## Official specification

- **llmstxt.org** (primary spec): https://llmstxt.org/
- **GitHub repository**: https://github.com/AnswerDotAI/llms-txt
- **Proposer**: Jeremy Howard (Answer.AI), September 2024
- **Status**: Community convention; not W3C/IETF; syntax stable across adopters as of 2026

## Spec structure (normative summary)

From https://llmstxt.org/#format — sections in order:

1. `# Title` (H1) — required
2. `> Blockquote` — short summary (recommended)
3. Free Markdown body — paragraphs/lists, no headings (optional)
4. `## Section` — H2 delimited file lists (zero or more)
5. `## Optional` — special section; links may be skipped for shorter context

Link item format:

```markdown
- [name](url): optional notes
```

## Markdown page mirrors

Spec proposal: LLM-readable Markdown at same URL + `.md`:

- `https://example.com/page.html` → `https://example.com/page.html.md`
- Extensionless: append `index.html.md`

Reference: https://llmstxt.org/#proposal

## Expanded context files

| File | Purpose |
|------|---------|
| `llms.txt` | Curated index (links + descriptions) |
| `llms-full.txt` | Index with inlined page content |
| `llms-ctx.txt` | Generated context (no optional URLs) |
| `llms-ctx-full.txt` | Generated context (includes optional URLs) |

Tool: **llms_txt2ctx** — https://llmstxt.org/ (Integrations section)

## Comparison with other standards

| Standard | Role |
|----------|------|
| [robots.txt](https://www.rfc-editor.org/rfc/rfc9309.html) | Crawl directives for automated agents |
| [sitemap.xml](https://www.sitemaps.org/protocol.html) | Complete URL enumeration for search indexing |
| **llms.txt** | Curated LLM-oriented overview and deep-link map |
| [JSON-LD / schema.org](https://schema.org/) | Structured page semantics (see json-ld skill) |

llms.txt does **not** embed robots.txt rules.

## HTTP delivery

- Path: `/llms.txt` (root canonical)
- Media type: `text/plain; charset=utf-8` (common practice)
- Cache: `public, s-maxage=3600, stale-while-revalidate=86400` or similar for dynamic routes
- Optional mirror: `/.well-known/llms.txt`

## Next.js patterns

### Static

`public/llms.txt` — zero config; served at `/llms.txt`.

### Dynamic route handler

```
app/llms.txt/route.ts   → GET returns Markdown string
```

Use server actions or CMS queries to populate blog/docs slugs. Return `Response` with `text/plain` charset.

### Route segment note

Folder name `llms.txt` is valid in App Router; exports `GET` handler.

## Authoring best practices (from spec)

Source: https://llmstxt.org/#example

- Concise, clear language
- Informative descriptions on every link
- Avoid ambiguous terms or unexplained jargon
- Expand with `llms_txt2ctx` and test with real LLM queries

## Integrations and tooling

From https://llmstxt.org/ (Integrations):

| Tool | URL / package |
|------|----------------|
| llms_txt2ctx | CLI + Python module — expand llms.txt to context |
| vitepress-plugin-llms | VitePress auto-generation |
| docusaurus-plugin-llms | Docusaurus plugin |
| llms-txt-php | PHP read/write library |
| Drupal LLM Support | Drupal 10.3+ recipe |
| JavaScript implementation | Sample parser (see GitHub repo) |
| VS Code PagePilot | Loads external docs context in chat |

## Directories and discovery

- llmstxt.org lists directories of sites publishing llms.txt (see site footer / directories section)
- Validator/generator tools: e.g. https://llmtxt.info/ (community reference, validator, generator)

## Real-world examples

- FastHTML: https://fastht.ml/llms.txt (reference implementation cited on llmstxt.org)
- Anthropic, Cloudflare, Stripe, Mintlify, Vercel docs (widely cited adopters)
- Zod: https://zod.dev/llms.txt — API index for agents

## Multilingual sites

Convention: separate files per locale root, e.g.:

- `https://example.com/en/llms.txt`
- `https://example.com/es/llms.txt`

Cross-link locale files in the body section if helpful.

## SEO / GEO notes

- No major search engine has publicly committed to ranking based on llms.txt alone (2026).
- Still valuable as **agent infrastructure** and documentation contract.
- Pair with sitemap, robots.txt, and JSON-LD for complete discoverability stack.

## Related skills

- **sitemap**: sitemaps.org protocol for search engine crawling.
- **json-ld**: schema.org structured data for rich results.
