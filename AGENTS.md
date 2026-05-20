# AGENTS.md

## Repository Purpose

- This repo is primarily a Karpathy-style LLM wiki, not an application codebase.
- Use the `karpathy-llm-wiki` skill for wiki ingest, query, archive, or lint work.
- Prefer the project-local skill at `.agents/skills/karpathy-llm-wiki/SKILL.md` when available, so wiki workflows remain available even without the global skill install.
- Treat `raw/` as immutable collected source material; write synthesized knowledge to `wiki/`.

## Wiki Workflow

- `wiki/index.md` is the navigation source of truth; read it before answering questions from the wiki.
- `wiki/log.md` is append-only history; update it after ingest, archive, or lint operations.
- Current compiled topic content lives under `wiki/tiktok/`.
- Current raw TikTok sources are nested under `raw/tiktok/`, including `shop/`, `affiliate/`, and `order-delivery/`.
- When adding or materially updating a wiki article, refresh its `Updated:` date and update the matching `wiki/index.md` row.
- Preserve source attribution in article frontmatter (`Sources:` and `Raw:`); do not replace source facts with model memory.
- For TikTok Shop policy/fee answers, prefer `wiki/` summaries first, then verify against the linked `raw/` source when details matter.

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
