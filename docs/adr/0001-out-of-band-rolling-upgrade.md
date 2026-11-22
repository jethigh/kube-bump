# 0001. Out-of-Band Rolling Upgrade Strategy via Ansible

- **Date:** 2026-11-22
- **Status:** Proposed

## Context
Upgrading kubeadm-managed Kubernetes clusters requires strict ordering: control plane before workers, sequential node drains, etcd backups, and host-level snapshotting. Running upgrade agents inside the cluster risks deadlocks when the control plane or CNI is restarting.

## Decision
Use Ansible as an external out-of-band orchestrator with `serial: 1` execution, delegating cluster administration commands (`drain`, `uncordon`) through the primary control plane while driving local hypervisor snapshots directly from the control node.

## Alternatives Considered
- **In-cluster operator (e.g., System Upgrade Controller):** Cannot capture pre-upgrade VirtualBox hypervisor snapshots and risks failure if the control plane API becomes unresponsive during self-upgrade.
- **Manual Bash shell scripts:** Lack declarative inventory management, idempotent package state handling (`hold`/`unhold`), and multi-host error isolation.

## Consequences
- **Positive:** Guarantees cluster safety via external host snapshots and etcd backups prior to mutation; maintains workload availability with enforced `serial: 1` node draining.
- **Negative / Trade-offs:** Requires an external control machine with Ansible installed, SSH connectivity, and passwordless sudo access to all cluster nodes.
