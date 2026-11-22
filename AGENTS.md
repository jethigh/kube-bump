---
trigger: always_on
description: Core project overview, commands, structure, conventions, ADR rules, Git workflow, and agent boundaries.
---

# Project Rules

## Commands
Run these exact commands from the repository root to verify work (overrides default commands in language-specific rules):
- Install: `uv sync`
- Syntax check: `ansible-playbook playbooks/upgrade.yml --syntax-check`
- Lint: `ansible-lint playbooks/upgrade.yml`
- YAML lint: `yamllint .`

## Project & Stack
- **Purpose:** Automated, zero-downtime rolling upgrade orchestrator for kubeadm-based Kubernetes clusters with integrated etcd backups and VirtualBox hypervisor live snapshots.
- **Stack (with versions):** Ansible >= 2.15, Python >= 3.10, Kubernetes (kubeadm, kubelet, kubectl) >= 1.30, VirtualBox (VBoxManage).
- **Structure:**
  - `playbooks/` — Ansible playbooks (`upgrade.yml`) and modular task includes (`tasks/update_k8s_repo.yml`).
  - `inventory/` — Cluster host definitions (`hosts.ini`) and configuration variables (`group_vars/all.yml`).
  - `docs/architecture.md` — Module boundaries, data flow, external integrations, and invariants.
  - `docs/adr/` — Architectural Decision Records (immutable records of architectural decisions).

## Testing & Code Style
- **Test convention:** Validate playbook syntax with `ansible-playbook --syntax-check playbooks/upgrade.yml` and enforce code standards via `ansible-lint`. Never execute playbooks against live clusters or inventories without explicit user authorization.
- **Canonical examples (follow these instead of inventing new patterns):**
  - Node upgrade sequence and task delegation: `playbooks/upgrade.yml` (e.g., node drain and uncordon delegated to `control_plane[0]`).
  - Dynamic APT repository and GPG key management: `playbooks/tasks/update_k8s_repo.yml`.
- **Gotchas / Non-obvious rules:**
  - Never execute `ansible-playbook` against live cluster hosts (even with `--check`) without explicit user permission.
  - Always use Fully Qualified Collection Names (FQCN) for modules (e.g., `ansible.builtin.apt`, `ansible.builtin.command`).
  - Kubernetes upgrades must strictly progress sequentially by exactly one minor version ($N \rightarrow N+1$); multi-version minor skips are unsupported by kubeadm.
  - Worker nodes must never be upgraded before control plane nodes reach the target version.
  - Package selections for `kubeadm`, `kubelet`, and `kubectl` must always be placed on `hold` after package upgrades.
  - Commands executing `kubectl` or `kubeadm` must specify explicit `changed_when` conditions to preserve idempotency and satisfy linter checks.

## Project Context & ADRs
Links below are not auto-loaded — read them explicitly when relevant:
- Read [docs/architecture.md](docs/architecture.md) before adding a new module, integration, or changing data flow.
- Check filenames in `docs/adr/` before altering core architecture, auth, storage, or dependencies; read matching ADRs.
- When making a significant architectural trade-off (only when requested or during a structural change), create a new ADR using [docs/adr/0000-template.md](docs/adr/0000-template.md). Never edit past ADRs; supersede them instead.
- Do not document file trees, line-by-line summaries, or rules already enforced by linters/formatters.

## Git Workflow
- Branches: `feat/<slug>`, `fix/<slug>`, `chore/<slug>`, `docs/<slug>`
- Commits: Conventional Commits: `feat(scope): short summary`, `fix(scope): short summary`, `docs(scope): short summary`
- Trailers: Include co-authorship trailer: `Co-authored-by: <Tool/Agent> <<identifier_or_email>>`

## Boundaries
- **Language:** In created files in comments, readme, documentations use English. With user communicate with language used by user.
- **Always do:** Run syntax check and lint after edits; keep diffs focused on the requested task.
- **Ask first:** Adding new dependencies, modifying cluster upgrade sequence, changing CI/CD workflows, or contradicting an accepted ADR.
- **Never do:** Run destructive commands or live cluster playbook execution without user consent; commit secrets or `.env` files; skip or weaken failing lint checks.
