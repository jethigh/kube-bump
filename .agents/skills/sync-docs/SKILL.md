---
name: sync-docs
description: Synchronize docs/architecture.md and create Architectural Decision Records (ADRs) in docs/adr/ after implementing a feature, refactor, or dependency change. Use when asked to update project docs, record an architectural decision, or wrap up a structural change.
---

# Sync Project Documentation & ADRs

Keep project documentation accurate and minimal based on actual code changes. Never pad docs with implementation details already visible in code.

## Step 1: Inspect Actual Changes
Determine what actually changed in the repository before reading or editing any docs:
1. Run `git status` and `git diff` (or `git diff main...HEAD` if changes are already committed on a feature branch).
2. Identify whether the change alters:
   - Module boundaries or responsibilities
   - Data flow, persistence, or public contracts
   - External integrations or infrastructure
   - Dependencies or non-obvious architectural trade-offs

## Step 2: Triage (Skip Gate)
Stop early if the changes are purely:
- Bug fixes that do not change contracts or architecture
- Routine CRUD/UI additions following existing patterns
- Formatting, renaming, or internal refactors inside a single module
- Test-only updates

If no architectural or decision-level change occurred, do NOT edit any files. Report: `No architecture or ADR updates needed for these changes.`

## Step 3: Update `docs/architecture.md` (If Needed)
Only if module boundaries, data flow, entry points, or external integrations changed:
1. Read `docs/architecture.md`.
2. Make surgical updates limited to the affected section.
3. Enforce these constraints:
   - Keep the entire file under 150 lines.
   - Document boundaries, flows, and invariants — never file trees, function lists, or line-by-line summaries.
   - Remove outdated descriptions instead of appending notes on top of them.

## Step 4: Create an ADR (Only for Real Trade-offs)
Create a new ADR only when a non-obvious decision was made (e.g., adding a core dependency, changing auth/storage/messaging strategy, introducing a new pattern, or reversing an earlier decision):
1. List all files in `docs/adr/` to determine the next sequential 4-digit number (`NNNN`) and check if any existing ADR is affected.
2. Read `docs/adr/0000-template.md` and follow its exact structure.
3. Create `docs/adr/NNNN-short-kebab-title.md`:
   - Set `Status:` to `Proposed` unless the user explicitly approved it as `Accepted`.
   - Keep each section brief and concrete (max ~40 lines total).
4. If this decision replaces an older ADR:
   - Edit ONLY the `Status:` line of the superseded ADR to `Superseded by [NNNN](NNNN-short-kebab-title.md)`.
   - Never rewrite the body of past ADRs.

## Step 5: Report
Summarize concisely:
- Which files were updated or created, with a 1-line reason per file.
- Any open architectural questions or assumptions that require human confirmation (especially if a new ADR is marked `Proposed`).