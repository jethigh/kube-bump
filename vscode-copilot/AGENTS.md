# AGENTS.md

## Commands
Run these exact commands from the repository root to verify work:
- Install: `[e.g., pnpm install]`
- Dev: `[e.g., pnpm dev]`
- Build: `[e.g., pnpm build]`
- Lint & Typecheck: `[e.g., pnpm lint && pnpm typecheck]`
- Test (all): `[e.g., pnpm test]`
- Test (single file): `[e.g., pnpm test -- path/to/file.test.ts]`

## Project & Stack
- **Purpose:** [1-2 sentences: what the project does]
- **Stack (with versions):** [e.g., TypeScript 5.x, Node 22, React 19, PostgreSQL 16]
- **Structure:**
  - `src/[module]/` — [responsibility]
  - `tests/` — [how tests mirror `src/`]
  - `docs/architecture.md` — Module boundaries, data flow, external integrations
  - `docs/adr/` — Architectural Decision Records (immutable)

## Testing & Code Style
- **Test convention:** [e.g., Vitest; mirror `src/` paths as `*.test.ts`; shared fixtures in `tests/fixtures/`]
- **Canonical examples (follow these instead of inventing new patterns):**
  - [e.g., API handler & error handling: `src/api/routes/users.ts`]
  - [e.g., Data access / repository: `src/db/repositories/users.ts`]
- **Gotchas / Non-obvious rules:**
  - [Add rules here ONLY after an agent makes the same mistake twice]

## Project Context & ADRs
Links below are not auto-loaded — read them explicitly when relevant:
- Read [docs/architecture.md](docs/architecture.md) before adding a new module, integration, or changing data flow.
- Check filenames in `docs/adr/` before altering core architecture, auth, storage, or dependencies; read matching ADRs.
- When making a significant architectural trade-off (only when requested or during a structural change), create a new ADR using [docs/adr/0000-template.md](docs/adr/0000-template.md). Never edit past ADRs; supersede them instead.
- Do not document file trees, line-by-line summaries, or rules already enforced by linters/formatters.

## Git Workflow
- Branches: `[e.g., feat/<slug>, fix/<slug>, chore/<slug>]`
- Commits: `[e.g., Conventional Commits: feat(scope): short summary]`

## Boundaries
- **Always do:** Run lint, typecheck, and targeted tests after edits; keep diffs focused on the requested task.
- **Ask first:** Adding new dependencies, modifying DB migrations/schema, changing CI/CD workflows, or contradicting an accepted ADR.
- **Never do:** Commit secrets or `.env` files; edit generated directories (`[e.g., src/generated/]`); skip or weaken failing tests.