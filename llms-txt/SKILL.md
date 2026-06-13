---
name: llms-txt
description: Create llms.txt per llmstxt.org for AI assistants and AEO/GEO. Use when adding llms.txt, llms-full.txt, AI-readable site maps, agent context, or when the user mentions llms.txt, llmstxt, answer engine optimization, GEO, or B2A content.
---

# llms.txt

## Overview

**AEO (Answer Engine Optimization)** covers how your site is understood and cited by AI assistants (ChatGPT, Perplexity, Claude, etc.). **`llms.txt`** is the primary convention: a Markdown file at `/llms.txt` that gives models a **curated map** of your most important pages.

It complements (does not replace) `robots.txt` and `sitemap.xml`. It is **not** a W3C/IETF standard; treat it as durable, machine-readable site documentation you control.

Reference: [llmstxt.org](https://llmstxt.org/) · [GitHub spec repo](https://github.com/AnswerDotAI/llms-txt)

---

## When to use llms.txt

| Use case | Why llms.txt helps |
|----------|-------------------|
| Product / SaaS docs | Agents find API guides, onboarding, pricing without crawling every page |
| Developer libraries | Point to `.md` doc URLs and quick-start examples |
| Marketing sites | Curate features, pricing, integrations for AI assistants |
| Personal / portfolio sites | Summarize CV, projects, contact in one fetch |

**Not a substitute for**: SEO (use sitemap + structured data), crawl policy (use `robots.txt`), or full-text search indexing.

---

## File location and delivery

- **Canonical path**: `https://example.com/llms.txt` (site root).
- **Optional**: subpath (e.g. `/docs/llms.txt`) or `/.well-known/llms.txt` (root is preferred).
- **Content-Type**: `text/plain; charset=utf-8` (Markdown body; plain text MIME is conventional).
- **Multilingual**: one file per language root (e.g. `/en/llms.txt`, `/es/llms.txt`).

### Static file

Place `public/llms.txt` in Next.js/Vite/static hosts. Served as-is at `/llms.txt`.

### Dynamic route (Next.js App Router)

When content comes from a CMS or database, use a route handler:

```ts
// app/llms.txt/route.ts
export async function GET() {
  const content = buildLlmsTxt(); // assemble Markdown string
  return new Response(content, {
    headers: {
      'Content-Type': 'text/plain; charset=utf-8',
      'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400',
    },
  });
}
```

Build the Markdown string in code; keep sections in spec order. Fetch dynamic slugs server-side (blog posts, docs) and append link lines.

---

## Spec format (required order)

Per [llmstxt.org](https://llmstxt.org/), sections appear **in this order**:

1. **H1** — project or site name (required; only mandatory section).
2. **Blockquote** — one short summary (`> ...`) with key context for interpreting the rest.
3. **Body** (optional) — paragraphs or lists with extra guidance; **no headings** in this block.
4. **H2 sections** — each section is a “file list” of markdown links.
5. **`## Optional`** (special) — secondary links agents may skip when context is limited.

### Link line syntax

Each entry is a markdown list item:

```markdown
- [Link title](https://absolute-url): Optional one-line description.
```

- `[name](url)` is required.
- `: description` is optional but strongly recommended.
- Use **absolute URLs**.
- Prefer links to **`.md` versions** of pages when available (same path + `.md`, or `index.html.md` for extensionless URLs).

### Minimal example

```markdown
# Acme Docs

> Acme is a task API for teams. REST + webhooks; auth via API keys.

Use the Quick Start first, then the API reference for endpoints.

## Docs

- [Quick start](https://docs.acme.com/quickstart.md): Install SDK and send your first request
- [API reference](https://docs.acme.com/api.md): All endpoints and error codes

## Examples

- [Node webhook handler](https://github.com/acme/examples/blob/main/webhook.ts): Verify signatures and process events

## Optional

- [Changelog](https://docs.acme.com/changelog.md): Release history
```

---

## Companion conventions

### Markdown mirrors (`.md` URLs)

Pages useful to LLMs should expose a clean Markdown copy at the **same URL + `.md`** (or `index.html.md`). Example: `https://docs.fastht.ml/docs/tutorials/quickstart_for_web_devs.html.md`.

### llms-full.txt / llms-ctx

- **llms-full.txt** — expanded file with page content inlined (single fetch for agents).
- **llms-ctx.txt** / **llms-ctx-full.txt** — generated via [`llms_txt2ctx`](https://llmstxt.org/) CLI from `llms.txt` (XML-ish bundle for tools like Claude).

Use when documentation is large; most marketing sites only need `llms.txt`.

---

## vs robots.txt and sitemap.xml

| File | Purpose |
|------|---------|
| **robots.txt** | Crawl permission / disallow rules for bots |
| **sitemap.xml** | Exhaustive list of indexable URLs for search engines |
| **llms.txt** | Curated, LLM-oriented overview with descriptions; may link externally |

llms.txt is aimed at **inference** (on-demand agent context), not training policy or full-site enumeration.

---

## Authoring guidelines

- **Concise** blockquote and descriptions; avoid marketing fluff agents cannot act on.
- **Curate** — link to 10–30 high-value pages, not every URL in the sitemap.
- **Stable URLs** — prefer canonical paths; update `llms.txt` when slugs change.
- **No ambiguous jargon** without a one-line explanation.
- **Test** — feed the file (or `llms-ctx` output) to an agent and verify it answers common questions.
- Put **secondary** links (changelog, legal, old posts) under `## Optional`.

---

## What to include (typical sections)

| H2 section | Typical links |
|------------|----------------|
| Main / Overview | Home, product, pricing |
| Docs | Getting started, API reference, guides |
| Blog / Updates | Key articles or category indexes |
| Integrations | SDKs, extensions, console/app |
| Optional | Changelog, legal, full archives |

Adapt section names to the site; only `## Optional` has special semantics in the spec.

---

## Framework integrations

Common generators (see [llmstxt.org integrations](https://llmstxt.org/)):

- **VitePress**: `vitepress-plugin-llms`
- **Docusaurus**: `docusaurus-plugin-llms`
- **Drupal**: LLM Support recipe
- **CLI**: `llms_txt2ctx` (Python) — parse and expand to context files
- **PHP**: `llms-txt-php` — read/write spec-compliant files

For custom stacks (Next.js + Sanity, etc.), a `route.ts` handler that builds Markdown from queries is the usual pattern.

---

## Common mistakes

- **Missing H1** — only required element; file is invalid without it.
- **Wrong section order** — blockquote before H2s; body before first H2.
- **Relative URLs** — use absolute `https://` links.
- **Dumping sitemap** — too many low-value links defeats the purpose.
- **Using llms.txt for robots rules** — use `robots.txt` instead.
- **Broken link format** — must be `- [Title](URL): description`; parsers expect this shape.
- **Skipping descriptions** — weak for agents choosing which link to fetch next.

---

## Related skills

- **sitemap** — exhaustive URL list for search crawlers (different purpose than llms.txt).
- **json-ld** — schema.org structured data for Google rich results.

---

## Additional resources

- [reference.md](reference.md) — Official spec, tools, directories, examples, comparisons.
- Spec: https://llmstxt.org/
- GitHub: https://github.com/AnswerDotAI/llms-txt
