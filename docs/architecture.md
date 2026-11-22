# Architecture

## System Overview
kube-bump is an out-of-band Ansible automation suite that orchestrates sequential, zero-downtime rolling upgrades for kubeadm-based Kubernetes clusters. The automation operates via SSH from an external control machine, driving node cordon/drain, package selection holds, kubeadm execution plans, and local host hypervisor snapshots.

## Module Boundaries
| Module / Path | Responsibility | Owns Data / State | Allowed Dependencies |
|---|---|---|---|
| `playbooks/upgrade.yml` | Main orchestration pipeline: executes pre-flight, control plane upgrades, and worker node rollouts. | None (stateless orchestrator). | `playbooks/tasks/update_k8s_repo.yml`, Ansible core modules. |
| `playbooks/tasks/update_k8s_repo.yml` | Dynamic package repository and GPG keyring provisioning for target Kubernetes minor versions. | `/etc/apt/keyrings`, `/etc/apt/sources.list.d/kubernetes.list`. | `curl`, `gpg`, `apt_repository`. |
| `inventory/` | Defines cluster topology, host addressing, hypervisor mapping, and global configuration variables. | Inventory host and group variables. | None. |

## Core Data Flows
- **Pre-flight Snapshots:** Local Control Node -> `VBoxManage` -> VirtualBox Hypervisor VM State
- **Disaster Recovery Backup:** Local Control Node -> `control_plane[0]` -> `etcdctl` -> `/var/backups/kube-bump/etcd-<epoch>.db`
- **Control Plane Upgrade:** Local Control Node -> `control_plane[0]` -> `kubeadm upgrade apply` -> `kubectl drain` -> `apt upgrade kubelet/kubectl` -> `kubectl uncordon`
- **Worker Rolling Upgrade:** Local Control Node -> Worker Nodes (`serial: 1`) -> `kubeadm upgrade node` -> `control_plane[0]` (delegated drain) -> `apt upgrade kubelet/kubectl` -> `control_plane[0]` (delegated uncordon)

## External Integrations & Infrastructure
- **VirtualBox (`VBoxManage`):** Executed locally on the control host to take live VM snapshots prior to host mutation. If snapshotting fails, the run halts before cluster disruption.
- **etcd (`etcdctl`):** Direct CLI interaction via PKI client certificates on `control_plane[0]` to snapshot Raft log and datastore state.
- **Kubernetes APT Repository (`pkgs.k8s.io`):** Network fetch of target minor version release key and repository configuration.
- **Kubernetes API Server (`kubectl`):** Administrative interactions via `/etc/kubernetes/admin.conf` for workload eviction (`drain`) and scheduling restoration (`uncordon`).

## Key Invariants
- Upgrades must proceed sequentially by exactly one minor version ($N \rightarrow N+1$); multi-version minor skips are strictly forbidden.
- Control plane nodes must always reach the target version before worker nodes are touched.
- All node drains and uncordons must be executed or delegated through the primary control plane admin credentials.
- Package manager selections for `kubeadm`, `kubelet`, and `kubectl` must remain on `hold` except during the active upgrade transaction.
- All worker node upgrades must execute with `serial: 1` to prevent concurrent multi-node workload evictions.
