# AGENTS.md

## Repository Purpose

- This repo is primarily a Karpathy-style LLM wiki, not an application codebase.
- Use the `karpathy-llm-wiki` skill for wiki ingest, query, archive, or lint work.
- Prefer the project-local skill at `.agents/skills/karpathy-llm-wiki/SKILL.md` when available, so wiki workflows remain available even without the global skill install.
- Treat `raw/` as immutable collected source material; write synthesized knowledge to `wiki/`.

## Wiki Workflow

- `wiki/index.md` is the top-level navigation map and should stay concise. Read it before answering questions from the wiki.
- `wiki/log.md` is append-only history; update it after ingest, archive, or lint operations.
- Domain index files such as `wiki/product/index.md` and `wiki/engineering/index.md` list topics in that domain.
- Topic index files such as `wiki/product/tiktok/index.md` list detailed article rows for that topic.
- Detailed article tables belong in topic-level indexes. Domain indexes should list topics, and the root index should list domains.
- Keep `raw/` simple and source-oriented. Store collected source material under source/entity buckets such as `raw/tiktok/`, `raw/whatsapp/`, `raw/xendit/`, `raw/bun/`, or `raw/papers/`.
- Organize compiled wiki articles by knowledge domain and topic using `wiki/<domain>/<topic>/<article>.md`.
- Primary wiki domains are `product/`, `engineering/`, `science/`, `business/`, and `design/`.
- Use `wiki/product/` for product behavior, platform policies, pricing, seller/user operations, workflows, and feature knowledge.
- Use `wiki/engineering/` for APIs, SDKs, protocols, webhooks, infrastructure, implementation details, code, and developer documentation.
- Use `wiki/science/` for scientific concepts, research papers, theories, experiments, and technical scientific notes.
- Use `wiki/business/` for markets, monetization, strategy, operations, business models, and competitive analysis.
- Use `wiki/design/` for UX, visual design, interaction patterns, brand systems, and product design principles.
- A single raw source bucket may feed multiple wiki domains. For example, `raw/whatsapp/` can produce both `wiki/product/whatsapp/` and `wiki/engineering/whatsapp/` articles.
- If placement confidence is not high, ask the human for confirmation before creating or moving a wiki article. Do not silently place ambiguous documents into a generic folder.
- Ambiguous placement examples that require confirmation include WhatsApp sources mixing pricing with API implementation, platform documents mixing business strategy with product operations, and scientific engineering papers that could fit both `science/` and `engineering/`.
- When adding or materially updating a wiki article, refresh its `Updated:` date and update the matching topic-level index row. If the topic or domain is new, also update the domain index and root index as needed.
- Preserve source attribution and source fidelity in article frontmatter and body. Keep `Sources:` and `Raw:` links, and retain important operational details instead of over-compressing raw material.
- For TikTok Shop policy/fee answers, prefer `wiki/` summaries first, then verify against the linked `raw/` source when details matter.

## Wiki Ingestion Fidelity

- Preserve thorough source detail during wiki ingestion. Do not over-simplify source documents into shallow summaries.
- Wiki articles should be synthesized and structured, but they must retain important operational details, edge cases, constraints, formulas, examples, tables, caveats, and source-specific distinctions from the raw material.
- Prefer clear organization over compression. If a source is detailed, the wiki article may also be detailed.
- Summaries are acceptable as entry points, but they must not replace the underlying details needed for future reasoning, implementation, policy checks, or decision-making.
- When compressing repeated content, preserve at least one representative example and note the repeated pattern or rule explicitly.
- Preserve source-specific disagreements, version/date differences, regional differences, pricing tiers, API shape details, limits, thresholds, eligibility rules, and exceptions.
- If ingestion would require heavy summarization because an article is getting too broad, split it into multiple focused wiki articles instead of dropping detail.
- Raw sources remain the immutable full record, but wiki articles should be detailed enough that common questions can be answered from `wiki/` first and raw files are only needed for verification or deep audit.

## Graphify Context

- `graphify-out/graph.json` and `graphify-out/GRAPH_REPORT.md` already exist; read `GRAPH_REPORT.md` before broad searches or architecture questions.
- Use Graphify query/navigation before `rg`, fuzzy search, or broad file scans; fall back to `rg`/Glob/Grep only when Graphify is unavailable or not initialized.
- `.opencode/opencode.json` loads `.opencode/plugins/graphify.js`, which only injects a one-time bash reminder when the graph exists.
- `graphify-out/` is persistent graph context and should remain git-trackable; update it with Graphify rather than hand-editing it.

## Project Commands

- Python version is pinned by `.python-version` and `pyproject.toml` to Python `3.13` / `>=3.13`.
- Use the repo-local `.venv` whenever creating or running Python code.
- Use `uv` instead of `pip` for Python dependency and execution workflows.
- Ask before installing new tools or dependencies; if approved, record Python dependencies/tools in `pyproject.toml` instead of installing them ad hoc.
- There are no configured test, lint, formatter, typecheck, or codegen commands in this repo.
- The only Python entrypoint is the scaffold `main.py`; run it with `python main.py` if needed.
- OpenCode plugin dependencies live in `.opencode/package.json`; if reinstalling them, run npm commands from `.opencode/`, not the repo root.

## Files To Ignore For Normal Wiki Work

- `.obsidian/` contains reusable vault config; ignore only local workspace/session files, not the whole directory.
- `.venv/`, `__pycache__/`, build outputs, and Python package artifacts are ignored by `.gitignore`.
- `README.md` is currently empty and is not a reliable source of repo behavior.
