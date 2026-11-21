---
trigger: glob
globs: "**/*.sh, **/*.bash"
description: Safe Bash scripting conventions and verification commands (bash -n, ShellCheck) for maintainable shell scripts.
---

# Bash Guidelines

Assume ShellCheck enforces quoting, `local` return masking, and syntax lints. Follow these runtime safety and design rules:

## Verification Commands
Unless project rules (`.agent/rules/project.md`) define different project-specific commands, run after editing any shell script:
- Syntax check: `bash -n path/to/script.sh`
- Lint: `shellcheck path/to/script.sh`
- Format (if installed): `shfmt -w -i 2 -ci path/to/script.sh`

## Runtime Safety & Structure
- Start every script with `#!/usr/bin/env bash` and `set -euo pipefail`.
- Handle expected non-zero exits explicitly (`if cmd; then` or `cmd || true`). Avoid `((i++))` when `i` can be `0` under `set -e`; use `((++i))`.
- Use `mktemp` / `mktemp -d` with `trap cleanup EXIT` for temporary files; never hardcode `/tmp/` paths.
- Send logs and errors to `stderr`, keeping `stdout` clean for pipeable data.
- Validate required external tools up front (`command -v tool >/dev/null 2>&1`) and fail with a clear message.
- Keep Bash for glue and orchestration; if a script grows beyond ~100 lines or requires complex JSON/data manipulation, propose rewriting it in Python instead.