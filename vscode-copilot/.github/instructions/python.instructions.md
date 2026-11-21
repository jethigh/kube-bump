---
description: Python design guidelines and standard verification commands (uv, Ruff, Pyright, pytest) for linting, formatting, typechecking, and running Python tests.
applyTo: "**/*.py,pyproject.toml"
---

# Python Guidelines

## Verification Commands
Unless `AGENTS.md` defines different project-specific commands, run these from the repository root after editing Python code:
- Auto-fix & format: `uv run ruff check --fix . && uv run ruff format .`
- Lint & format check: `uv run ruff check . && uv run ruff format --check .`
- Typecheck: `uv run pyright`
- Test (single file): `uv run pytest tests/path/to/test_file.py`
- Test (all): `uv run pytest`

## Data Modeling & Interfaces
- Model domain data with `@dataclass(frozen=True, slots=True)`, `TypedDict`, or `Enum` instead of unstructured nested `dict`s.
- Accept abstract read-only collections in function parameters (`Sequence[T]`, `Mapping[K, V]`, `Iterable[T]`) when you do not mutate inputs; return concrete types (`list`, `dict`).
- Use `Any` only at truly untyped external boundaries, and validate/narrow it immediately upon entry.
...