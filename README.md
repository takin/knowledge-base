# Knowledges

This repository is an Obsidian-friendly personal knowledge vault maintained as a Karpathy-style LLM wiki.

The core idea is simple: raw source material is preserved, and synthesized knowledge is compiled into structured wiki articles that compound over time.

## Structure

```text
raw/
  <source-or-entity>/

wiki/
  index.md
  log.md
  product/
    index.md
    <topic>/
      index.md
      <article>.md
  engineering/
    index.md
    <topic>/
      index.md
      <article>.md
  science/
    index.md
  business/
    index.md
  design/
    index.md
```

## Raw Sources

`raw/` stores collected source material by source or entity, not by final knowledge category.

Examples:

```text
raw/tiktok/
raw/whatsapp/
raw/xendit/
raw/bun/
raw/papers/
```

Treat `raw/` as immutable source evidence. Do not rewrite source facts during ingestion.

## Compiled Wiki

`wiki/` stores synthesized knowledge by domain and topic:

| Domain | Use For |
| --- | --- |
| `product/` | Product behavior, platform policies, pricing, seller/user operations, workflows, and feature knowledge. |
| `engineering/` | APIs, SDKs, protocols, webhooks, infrastructure, implementation details, code, and developer documentation. |
| `science/` | Scientific concepts, research papers, theories, experiments, and technical scientific notes. |
| `business/` | Markets, monetization, strategy, operations, business models, and competitive analysis. |
| `design/` | UX, visual design, interaction patterns, brand systems, and product design principles. |

A single raw source bucket can feed multiple wiki domains. For example, `raw/whatsapp/` can produce both `wiki/product/whatsapp/` and `wiki/engineering/whatsapp/` articles.

## Index System

The index hierarchy is intentionally split so the vault can grow without making the root index too large.

- `wiki/index.md` is the top-level domain map.
- `wiki/<domain>/index.md` lists topics in that domain.
- `wiki/<domain>/<topic>/index.md` lists detailed article rows for that topic.
- Detailed article tables belong in topic-level indexes, not the root index.

Start navigation from [`wiki/index.md`](wiki/index.md).

## Ingestion Rules

When ingesting new material:

1. Store raw material under the simplest source/entity bucket in `raw/`.
2. Classify the compiled article into `wiki/<domain>/<topic>/<article>.md` based on content.
3. If placement confidence is not high, ask a human before creating or moving the wiki article.
4. Preserve source attribution with `Sources:` and `Raw:` links in article frontmatter.
5. Update the article `Updated:` date and the matching topic-level index row.
6. If a new topic or domain is created, update the domain index and root index as needed.
7. Append important ingest, archive, lint, or maintenance operations to `wiki/log.md`.

## Fidelity Standard

Wiki articles should be synthesized and structured, but not over-simplified.

Preserve important operational details from raw sources, including edge cases, constraints, formulas, examples, tables, caveats, regional differences, version/date differences, pricing tiers, API shapes, limits, thresholds, eligibility rules, and exceptions.

If a source is too broad to summarize faithfully in one article, split it into multiple focused wiki articles instead of dropping detail.

## Agent Notes

Detailed agent instructions live in [`AGENTS.md`](AGENTS.md). Use those rules as the source of truth for automated ingestion, query, archive, and maintenance workflows.
