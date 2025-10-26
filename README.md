<h1 align="center">Haven</h1>

**A *forever-work-in-progress* self-hosted server setup**

Runs on a multi-node k3s cluster deployed across VMs and bare-metal hosts.

Application configuration is stored on an NFS share located on a dedicated SSD. This uses `nfs-subdir-external-provisioner` as a dynamic storage provisioner with PVC-specific paths. Additional data is stored on a NAS exported via NFS.

The cluster runs `k3s` with `nginx` as the ingress controller. `MetalLB` is used in layer 2 mode for load balancing. `cert-manager` provides a local CA and issues certificates (required by Vaultwarden).

For setup details, see [SETUP.md](SETUP.md).

The repository name references my local TLD, `.haven` ;)

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
    - Beszel Agent (running as a DaemonSet)
    - Code Config (VS Code for internal config editing)
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

- lab (a playground/sandbox namespace)
    - nfs-pod (for testing and accessing NFS mounts)
- metallb-system
    - MetalLB components
- cert-manager
    - cert-manager components

## Todo
- Move ArchiveBox data to its own PVC on the NAS
- Move Uptime Kuma to the infra namespace
- Add links to each application's documentation
- Add links to server scripts
- Move Alloy to the monitoring namespace
- Install Loki, Grafana, and Prometheus via Helm charts
- Configure Loki and Prometheus to use PVCs