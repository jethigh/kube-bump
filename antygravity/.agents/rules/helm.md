---
trigger: glob
globs: "**/Chart.yaml, **/values*.yaml, **/templates/**/*.yaml, **/templates/**/*.yml, **/templates/**/*.tpl"
description: Helm chart conventions, Go template safety rules, Kubernetes security defaults, and verification commands (helm lint, helm template, kubeconform).
---

# Helm Chart Guidelines

## Verification Commands
Unless project rules (`.agent/rules/project.md`) define different project-specific commands, run from the repository root after editing any chart file (replace `<chart-dir>` with the chart path):
- Lint: `helm lint --strict <chart-dir>`
- Render default values: `helm template test-release <chart-dir> >/dev/null`
- Render non-default branches: when adding/changing conditional blocks (`if`), also run `helm template test-release <chart-dir> --set <flag>=true >/dev/null`
- Validate K8s schemas (if `kubeconform` is installed): `helm template test-release <chart-dir> | kubeconform -strict -summary`

## Template Idioms & Safety
- Serialize nested maps/lists with `toYaml ... | nindent N` instead of manually iterating and hardcoding spaces; always inspect rendered YAML after changing `{{-` whitespace trimming.
- Centralize naming, standard Kubernetes labels (`app.kubernetes.io/*`), and selector labels in `templates/_helpers.tpl`.
- Never alter `spec.selector.matchLabels` on existing Deployments, StatefulSets, or DaemonSets without warning — Kubernetes selectors are immutable and break `helm upgrade`.
- Use `required "message" .Values.x` for mandatory values with no safe default instead of silently rendering empty strings.
- When a Pod depends on a templated ConfigMap or Secret that should trigger a rollout on change, add a `checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}` annotation to the Pod template.

## Values, Schema & Security Defaults
- Document every new key in `values.yaml` with a concise comment and update `values.schema.json` if the chart uses one.
- Never embed plaintext secrets, tokens, or default passwords in `values.yaml` or templates; support referencing an existing Kubernetes Secret (`existingSecret`).
- Never default image tags to `latest`; default to `.Chart.AppVersion` or an explicit pinned tag in `values.yaml`.
- Expose configurable `resources`, `podSecurityContext`, and `securityContext` in `values.yaml`, defaulting to least privilege (`runAsNonRoot: true`, `allowPrivilegeEscalation: false`, dropping `ALL` capabilities) where compatible with the workload.