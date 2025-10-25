<h1 align="center">Haven</h1>

**A *forever-work-in-progress* self-hosted server setup**

Based on a multi-node k3s cluster running on VMs and bare metal hardware.

The overall application configs are stored in a NFS share inside of a SSD that was purposed specifically for this. For that I'm using `nfs-subdir-external-provisioner` as a dynamic storage provisioner with specified paths on each PVC. Some other data is stored on a NAS server with a NFS share as well.

The cluster is running on `k3s` with `nginx` as the ingress controller. For load balancing I'm using `MetalLB` in layer 2 mode. I'm also using `cert-manager` for local CA and certificates (as Vaultwarden requires it).

For more information on setup, check out [SETUP.md](SETUP.md).

Also, the repository name is a reference to my local TLD which is `.haven` :)

## Namespaces
- default
    - ArchiveBox
    - Homarr
    - Homepage
    - It-tools
    - Notepad
    - Searxng
    - Uptimekuma
    - Vaultwarden
- dns
    - AdGuardHome
    - AdGuardHome-2 (2nd instance)
    - AdGuard-Sync
- infra
    - [Haven Notify](https://git.ivanch.me/ivanch/server-scripts/src/branch/main/haven-notify)
    - Beszel
    - Beszel Agent (running as DaemonSet)
    - Code Config (vscode for internal config editing)
    - WireGuard Easy
- dev
    - Gitea Runner (x64)
    - Gitea Runner (arm64)
- monitoring
    - Grafana
    - Prometheus
    - Node Exporter
    - Kube State Metrics
    - Loki
    - Alloy

#### Miscellaneous namespaces

- lab (A playground/sandbox namespace)
    - nfs-pod (for testing and accessing NFS mounts through NFS)
- metallb-system
    - MetalLB components
- cert-manager
    - Cert-Manager components

## Todo:
- Move archivebox data to its own PVC on NAS
- Move uptimekuma to `infra` namespace
- Add links to each application docs
- Add links to server scripts
- Move alloy to `monitoring` namespace
- Change `loki`, `grafana`, and `prometheus` to Helm chart installation
- Change loki and prometheus to use PVCs