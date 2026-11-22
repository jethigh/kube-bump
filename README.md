<p align="center">
  <img src="assets/logo.png" alt="kube-bump logo" width="220" />
</p>

<h1 align="center">kube-bump</h1>

<p align="center">
  Automated rolling upgrade orchestrator for Kubernetes clusters (kubeadm) using Ansible and VirtualBox snapshot integration.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white" alt="Ansible" />
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes" />
  <img src="https://img.shields.io/badge/VirtualBox-183A61?style=for-the-badge&logo=virtualbox&logoColor=white" alt="VirtualBox" />
</p>

---

## Overview

kube-bump is an Ansible playbook suite designed to automate end-to-end, zero-downtime rolling upgrades of Kubernetes clusters initialized via `kubeadm`. The automation adheres strictly to the official Kubernetes upgrade lifecycle and version-skew policies, managing package repositories, service binaries, etcd state backups, and host-level VirtualBox virtual machine snapshots.

---

## Features

- **Automated VirtualBox Snapshots:** Executes live VM snapshots via `VBoxManage` from the control host prior to applying changes.
- **etcd Disaster Recovery Backup:** Generates cryptographically authenticated etcd database snapshots on the primary control plane node before control plane binary upgrades.
- **Strict Rolling Upgrade (`serial: 1`):** Sequentially cordons, drains, upgrades, and uncordons nodes to maintain workload availability.
- **Dynamic Repository Management:** Reconfigures `pkgs.k8s.io` apt repositories and GPG signing keyrings dynamically for the target minor version.
- **Package Selection Locking:** Manages `dpkg` selections (`hold`/`unhold`) for `kubeadm`, `kubelet`, and `kubectl` to prevent uncoordinated package manager upgrades.

---

## How It Works

The upgrade workflow in `playbooks/upgrade.yml` executes across four sequential phases:

```
[Phase 1: Localhost]
       |
       +--> VirtualBox Live Snapshots (all cluster VMs)
       |
[Phase 2: Primary Control Plane (control_plane[0])]
       |
       +--> etcd snapshot backup (/var/backups/kube-bump)
       +--> Update apt repository (pkgs.k8s.io for target minor version)
       +--> Upgrade kubeadm binary
       +--> kubeadm upgrade apply v<target_version>
       +--> Drain node (evict workloads)
       +--> Upgrade kubelet and kubectl binaries
       +--> Restart kubelet service
       +--> Uncordon node
       |
[Phase 3: Secondary Control Planes (control_plane[1:]) - serial: 1]
       |
       +--> Update apt repository
       +--> Upgrade kubeadm binary
       +--> kubeadm upgrade node
       +--> Drain node (delegated to control_plane[0])
       +--> Upgrade kubelet and kubectl binaries
       +--> Restart kubelet service
       +--> Uncordon node (delegated to control_plane[0])
       |
[Phase 4: Worker Nodes (workers) - serial: 1]
       |
       +--> Update apt repository
       +--> Upgrade kubeadm binary
       +--> kubeadm upgrade node
       +--> Drain node (delegated to control_plane[0])
       +--> Upgrade kubelet and kubectl binaries
       +--> Restart kubelet service
       +--> Uncordon node (delegated to control_plane[0])
```

### 1. Phase 1: Local Pre-flight and Infrastructure Snapshots
Executed on the Ansible control machine (`hosts: localhost`, `connection: local`). If `vbox_snapshot_enabled` is set to `true`, the playbook iterates over all hosts in `k8s_cluster` and issues:
```bash
VBoxManage snapshot "<vbox_vm_name>" take "<snapshot_name>" --live
```
Snapshot names follow the convention: `<prefix>-<k8s_target_version>-<epoch>`.

### 2. Phase 2: Primary Control Plane Upgrade
Targeted at `control_plane[0]`:
1. **etcd Backup:** If `etcd_backup_enabled: true`, issues `etcdctl snapshot save` against `https://127.0.0.1:2379` using cluster certificates located in `/etc/kubernetes/pki/etcd/`.
2. **Repository Switch:** Downloads the release key from `https://pkgs.k8s.io/core:/stable:/v<minor>/deb/Release.key` and writes `/etc/apt/sources.list.d/kubernetes.list`.
3. **kubeadm Upgrade:** Unholds `kubeadm`, installs `kubeadm=<k8s_target_version>-*`, and restores package hold.
4. **Cluster Plan Application:** Executes `kubeadm upgrade apply v<k8s_target_version> -y` to upgrade static control plane pod manifests and cluster configurations.
5. **Node Eviction:** Executes `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data` via `/etc/kubernetes/admin.conf`.
6. **Kubelet & Kubectl Upgrade:** Upgrades `kubelet` and `kubectl`, re-holds package versions, executes `systemctl daemon-reload`, and restarts `kubelet`.
7. **Node Restoration:** Executes `kubectl uncordon <node>`.

### 3. Phase 3: Secondary Control Plane Upgrades
Targeted at `control_plane[1:]` (for High Availability setups) with `serial: 1`:
1. Updates apt repository and keys for the target minor version.
2. Upgrades `kubeadm` binary.
3. Executes `kubeadm upgrade node` to update local static pod configurations.
4. Drains the node via command delegated to `control_plane[0]`.
5. Upgrades `kubelet` and `kubectl` binaries, then restarts `kubelet`.
6. Uncordons the node via command delegated to `control_plane[0]`.

### 4. Phase 4: Worker Node Rolling Upgrades
Targeted at `workers` with `serial: 1` to ensure zero cluster downtime:
1. Updates apt repository and keys for the target minor version.
2. Upgrades `kubeadm` binary.
3. Executes `kubeadm upgrade node` to refresh node configuration.
4. Drains worker workloads via command delegated to `control_plane[0]`.
5. Upgrades `kubelet` and `kubectl` binaries, then restarts `kubelet`.
6. Uncordons the worker node via command delegated to `control_plane[0]`.

---

## Prerequisites

### Control Machine
- Ansible `>= 2.15`
- Python `>= 3.10`
- OpenSSH client with key-based authentication configured
- `VBoxManage` available in `$PATH` (required only if `vbox_snapshot_enabled: true`)

### Managed Cluster Nodes
- Operating System: Ubuntu / Debian
- Architecture: amd64 / arm64
- Kubernetes deployment initialized with `kubeadm`
- Static IP addressing configured on each node
- User account with passwordless sudo access (or use `-K` flag during execution)
- `etcd-client` package installed on `control_plane[0]` (required only if `etcd_backup_enabled: true`)

---

## Configuration & Parameterization

### Global Variables

Global defaults are defined in `inventory/group_vars/all.yml` and can be overridden via inventory files or CLI extra variables (`-e` / `--extra-vars`).

| Parameter | Type | Default | Description |
|---|---|---|---|
| `k8s_target_version` | string | `"1.32.2"` | Explicit Kubernetes target version. Must only be incremented by one minor release at a time (e.g., `1.31.x` to `1.32.x`). |
| `k8s_minor_version` | string | `{{ k8s_target_version.split('.')[0:2] \| join('.') }}` | Computed minor version string (e.g., `"1.32"`), used to identify the official `pkgs.k8s.io` repository and release key. |
| `vbox_snapshot_enabled` | boolean | `true` | Toggles automatic creation of live VirtualBox VM snapshots from localhost before upgrade operations. |
| `vbox_snapshot_prefix` | string | `"pre-kube-bump"` | Prefix applied to the generated snapshot name. |
| `etcd_backup_enabled` | boolean | `true` | Toggles automated `etcd` snapshot creation on `control_plane[0]` before control plane upgrade. |
| `etcd_backup_dir` | string | `"/var/backups/kube-bump"` | Absolute directory path on `control_plane[0]` where etcd snapshots are stored. |

### Inventory Variables

Configured in `inventory/hosts.ini`:

| Variable | Scope | Required | Description |
|---|---|---|---|
| `ansible_host` | Host | Yes | Network IPv4 address or resolvable hostname of the node. |
| `vbox_vm_name` | Host | Conditional | Name of the virtual machine in VirtualBox. Required when `vbox_snapshot_enabled: true`. |
| `ansible_user` | Group / Host | Yes | Remote SSH username for Ansible connections. |
| `ansible_ssh_private_key_file` | Group / Host | Yes | Absolute or relative path to the private SSH key used for authentication. |

### Inventory Structure

```ini
[control_plane]
k8s-cp-1 ansible_host=192.168.1.50 vbox_vm_name="k8s-cp-1"

[workers]
k8s-worker-1 ansible_host=192.168.1.51 vbox_vm_name="k8s-worker-1"
k8s-worker-2 ansible_host=192.168.1.52 vbox_vm_name="k8s-worker-2"

[k8s_cluster:children]
control_plane
workers

[k8s_cluster:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

---

## Execution Guide

### 1. Verify Connectivity
Test SSH reachability and privilege escalation across all nodes:
```bash
ansible k8s_cluster -m ping
```

### 2. Verify Playbook Syntax
Run syntax validation against the playbook:
```bash
ansible-playbook playbooks/upgrade.yml --syntax-check
```

### 3. Run Standard Upgrade
Execute the complete upgrade pipeline using defaults defined in `inventory/group_vars/all.yml`:
```bash
ansible-playbook playbooks/upgrade.yml
```

### 4. Parameterized Execution (CLI Extra Vars)

Override configuration settings dynamically at runtime using the `-e` / `--extra-vars` flag:

#### Target a Specific Version
```bash
ansible-playbook playbooks/upgrade.yml -e "k8s_target_version=1.32.2"
```

#### Disable VirtualBox Snapshots (Bare-Metal or Cloud Deployments)
When running on physical hardware, cloud instances, or hypervisors other than VirtualBox, disable the snapshot task:
```bash
ansible-playbook playbooks/upgrade.yml -e "vbox_snapshot_enabled=false"
```

#### Disable etcd Backup
```bash
ansible-playbook playbooks/upgrade.yml -e "etcd_backup_enabled=false"
```

#### Set Custom etcd Backup Path
```bash
ansible-playbook playbooks/upgrade.yml -e "etcd_backup_dir=/opt/backups/k8s"
```

#### Combined Runtime Parameterization
```bash
ansible-playbook playbooks/upgrade.yml \
  -e "k8s_target_version=1.32.2" \
  -e "vbox_snapshot_enabled=false" \
  -e "etcd_backup_enabled=true" \
  -e "etcd_backup_dir=/var/backups/custom"
```

#### Privilege Escalation Prompt
If sudo requires a password on target nodes:
```bash
ansible-playbook playbooks/upgrade.yml -K
```

#### Explicit Inventory Path
If using a custom inventory file location:
```bash
ansible-playbook -i inventory/hosts.ini playbooks/upgrade.yml
```

#### Limit Execution Scope
To restrict execution to worker nodes (only permissible if control plane nodes are already at the target version):
```bash
ansible-playbook playbooks/upgrade.yml --limit workers
```

---

## Operational Constraints

- **Kubernetes Version Skew Policy:** Upgrades must proceed sequentially by minor version. Direct upgrades skipping minor versions (e.g., `v1.30.x` directly to `v1.32.x`) are unsupported by `kubeadm` and will cause validation errors.
- **Node Skew Order:** Control plane nodes must always be upgraded prior to worker nodes. Worker nodes must never run a higher minor version than the control plane components.
- **Pod Disruption Budgets (PDB):** Ensure that cluster workloads with active PDBs permit node draining, or workloads will block the drain process until timeouts occur.
