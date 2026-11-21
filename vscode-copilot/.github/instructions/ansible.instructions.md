---
description: Ansible playbook and role conventions, idempotency, check-mode safety, security defaults, and verification commands (ansible-lint, syntax-check).
applyTo: "**/ansible.cfg,**/playbooks/**/*.yml,**/playbooks/**/*.yaml,**/roles/**/*.yml,**/roles/**/*.yaml,**/roles/**/*.j2,**/group_vars/**,**/host_vars/**,**/inventories/**,**/inventory/**"
---

# Ansible Guidelines

Assume `ansible-lint` and `yamllint` enforce YAML formatting, FQCN module names, task naming, and basic lints. Follow these verification steps, runtime safety, and design rules:

## Verification Commands
Unless `AGENTS.md` defines different project-specific commands, run from the repository root after editing Ansible files:
- Lint: `ansible-lint`
- Syntax check: `ansible-playbook --syntax-check <playbook.yml>`
- YAML lint (if configured separately): `yamllint .`
- Never execute `ansible-playbook` against live hosts or inventories (even with `--check`) without explicit user confirmation.

## Idempotency & Check Mode
- Prefer native idempotent modules (`ansible.builtin.*`, official collections) over `command` or `shell`.
- When `command`/`shell` is unavoidable, use `creates`/`removes` or accurate `changed_when` and `failed_when` expressions — never add `changed_when: false` to a state-mutating command just to silence `ansible-lint`.
- Keep playbooks safe for `--check` mode: mark read-only discovery tasks needed by downstream `when:` conditions with `check_mode: false`, and guard tasks referencing registered variables that may be skipped in check mode.

## Security, Secrets & Config Safety
- Always set `no_log: true` on tasks handling passwords, tokens, private keys, or API secrets.
- Never commit plaintext credentials; use Ansible Vault variables or external secret lookups.
- Scope `become: true` as narrowly as possible (task or `block` level) instead of enabling it globally when only some tasks require root.
- When managing critical configs (`sshd`, `sudoers`, `nginx`, `systemd`), use `ansible.builtin.template` with `validate:` and explicit `owner`/`group`/`mode`, and trigger restarts/reloads via `handlers` (`notify`).

## Roles & Variables
- Prefix role variables with `<role_name>_` to prevent global namespace collisions.
- Put user-overridable defaults in `defaults/main.yml` and internal constants in `vars/main.yml`.
- Keep roles reusable; never hardcode environment-specific hostnames, IPs, or inventory group names inside roles.
