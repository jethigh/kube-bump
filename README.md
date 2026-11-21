<p align="center">
  <img src="assets/logo.png" alt="kube-bump logo" width="220" />
</p>

<h1 align="center">kube-bump</h1>

<p align="center">
  Automatyczny, bezpieczny rolling upgrade klastrów Kubernetes (<code>kubeadm</code>) z Ansible oraz opcjonalnymi snapshotami VirtualBox.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white" alt="Ansible" />
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes" />
  <img src="https://img.shields.io/badge/VirtualBox-183A61?style=for-the-badge&logo=virtualbox&logoColor=white" alt="VirtualBox" />
  <a href="https://github.com/<TWOJ_NICK>/kube-bump/actions">
    <img src="https://img.shields.io/github/actions/workflow/status/<TWOJ_NICK>/kube-bump/lint.yml?style=for-the-badge&label=ansible-lint" alt="Lint status" />
  </a>
</p>

---

## Funkcje

- **Snapshoty VirtualBox przed startem** — automatyczne wykonanie `--live` snapshotów przez `VBoxManage` przed modyfikacją klastra (można wyłączyć w `group_vars/all.yml`).
- **Backup `etcd`** — zrzut bazy `etcd` na głównym węźle Control Plane przed wywołaniem `kubeadm upgrade`.
- **Rolling upgrade (`serial: 1`)** — węzły są opróżniane (`drain`), aktualizowane i przywracane (`uncordon`) pojedynczo.
- **Zarządzanie pakietami `pkgs.k8s.io`** — automatyczne przełączanie repozytorium `apt` oraz `hold/unhold` dla `kubeadm`, `kubelet` i `kubectl`.

## Wymagania

- Ansible `>= 2.15` na maszynie sterującej.
- Dostęp SSH po kluczu oraz uprawnienia `sudo` (bez hasła lub z flagą `-K`) na węzłach klastra.
- Klaster `kubeadm` na Ubuntu/Debian ze statycznymi adresami IP (np. rezerwacja DHCP w sieci Bridge).
- `etcd-client` zainstalowany na głównym węźle Control Plane (jeśli `etcd_backup_enabled: true`).
- `VBoxManage` w `$PATH` na hoście (jeśli `vbox_snapshot_enabled: true`).

## Szybki start

1. **Uzupełnij `inventory/hosts.ini`** swoimi adresami IP oraz nazwami maszyn w VirtualBox.
2. **Sprawdź łączność:**
   ```bash
   ansible k8s_cluster -m ping