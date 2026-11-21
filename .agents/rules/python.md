---
trigger: glob
globs: "**/*.py, pyproject.toml"
description: Python design guidelines and standard verification commands (uv, Ruff, Pyright, pytest) for linting, formatting, typechecking, and running Python tests.
---

# Python Guidelines

Assume Ruff (strict ruleset) and the project typechecker enforce syntax, imports, `pathlib`, modern typing syntax, and security lints. Follow these verification steps and design rules:

## Verification Commands
Unless project rules (`.agent/rules/project.md`) define different project-specific commands, run these from the repository root after editing Python code:
- Auto-fix & format: `uv run ruff check --fix . && uv run ruff format .`
- Lint & format check: `uv run ruff check . && uv run ruff format --check .`
- Typecheck: `uv run pyright`
- Test (single file): `uv run pytest tests/path/to/test_file.py`
- Test (all): `uv run pytest`

## Data Modeling & Interfaces
- Model domain data with `@dataclass(frozen=True, slots=True)`, `TypedDict`, or `Enum` instead of unstructured nested `dict`s.
- Accept abstract read-only collections in function parameters (`Sequence[T]`, `Mapping[K, V]`, `Iterable[T]`) when you do not mutate inputs; return concrete types (`list`, `dict`).
- Use `Any` only at truly untyped external boundaries, and validate/narrow it immediately upon entry.

## Architecture, State & Resources
- Keep imports side-effect free: never perform I/O, network calls, or heavy initialization at module import time.
- Avoid mutable module-level globals and hidden singletons; pass dependencies (clients, repositories, config) explicitly so code stays testable.
- Manage all long-lived resources (HTTP clients, DB connections, locks, temp dirs) with context managers (`with` / `async with`), not only files.
- Guard executable entry-point scripts with `if __name__ == "__main__":`.

## Control Flow, Errors & Readability
- Use exceptions for exceptional failures, not routine branching. Define a small domain exception hierarchy rather than raising generic `Exception` or `RuntimeError` everywhere.
- Never use `assert` to validate runtime data or satisfy the typechecker; raise explicit `ValueError` or `TypeError`.
- Keep comprehensions simple (at most one `for` and one `if`); rewrite nested or multi-clause comprehensions as explicit loops.
- Write concise Google-style docstrings only for public APIs or non-obvious behavior; do not add docstrings that merely restate the function name and types.