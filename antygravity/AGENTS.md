---
trigger: always_on
description: Core project overview, commands, structure, conventions, ADR rules, Git workflow, and agent boundaries.
---

# Project Rules

## Commands
Run these exact commands from the repository root to verify work (overrides default commands in language-specific rules):
- Install: `[e.g., uv sync]`
- Dev: `[e.g., uv run ...]`
- Build: `[e.g., ...]`
- Lint & Typecheck: `[e.g., uv run ruff check . && uv run pyright]`
- Test (all): `[e.g., uv run pytest]`
- Test (single file): `[e.g., uv run pytest tests/path/to/test_file.py]`

## Project & Stack
- **Purpose:** [1-2 sentences: what the project does]
- **Stack (with versions):** [e.g., Python 3.12, Bash 5.2, Helm 3.x]
- **Structure:**
  - `src/[module]/` — [responsibility]
  - `tests/` — [how tests mirror `src/`]
  - `docs/architecture.md` — Module boundaries, data flow, external integrations
  - `docs/adr/` — Architectural Decision Records (immutable)

## Testing & Code Style
- **Test convention:** [e.g., pytest; mirror `src/` paths as `test_*.py`; shared fixtures in `tests/conftest.py`]
- **Canonical examples (follow these instead of inventing new patterns):**
  - [e.g., Core service & error handling: `src/app/services/example.py`]
  - [e.g., CLI / entry point: `src/app/cli.py`]
- **Gotchas / Non-obvious rules:**
  - [Add rules here ONLY after an agent makes the same mistake twice]

## Project Context & ADRs
Links below are not auto-loaded — read them explicitly when relevant:
- Read [docs/architecture.md](../../docs/architecture.md) before adding a new module, integration, or changing data flow.
- Check filenames in `docs/adr/` before altering core architecture, auth, storage, or dependencies; read matching ADRs.
- When making a significant architectural trade-off (only when requested or during a structural change), create a new ADR using [docs/adr/0000-template.md](../../docs/adr/0000-template.md). Never edit past ADRs; supersede them instead.
- Do not document file trees, line-by-line summaries, or rules already enforced by linters/formatters.

## Git Workflow
- Branches: `[e.g., feat/<slug>, fix/<slug>, chore/<slug>]`
- Commits: `[e.g., Conventional Commits: feat(scope): short summary]`

## Boundaries
- **Always do:** Run lint, typecheck, and targeted tests after edits; keep diffs focused on the requested task.
- **Ask first:** Adding new dependencies, modifying DB migrations/schema, changing CI/CD workflows, or contradicting an accepted ADR.
- **Never do:** Commit secrets or `.env` files; edit generated directories (`[e.g., src/generated/]`); skip or weaken failing tests.