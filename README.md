<h1 align="center">Haven</h1>

**A *forever-work-in-progress* self-hosted server setup**

Runs on a multi-node k3s cluster deployed across VMs and bare-metal hosts.

Application configuration is stored on an NFS share located on a dedicated SSD. This uses `nfs-subdir-external-provisioner` as a dynamic storage provisioner with PVC-specific paths. Additional data is stored on a NAS exported via NFS.

The cluster runs `k3s` with `nginx` as the ingress controller. `MetalLB` is used in layer 2 mode for load balancing. `cert-manager` provides a local CA and issues certificates (required by Vaultwarden).

For setup details, see [SETUP.md](SETUP.md).

The repository name references my local TLD, `.haven` ;)

## Repository Layout
```
haven/
├── bootstrap/
│   ├── namespaces.yaml        # cluster namespaces
│   ├── secretstores.yaml      # cluster secret stores for BitWarden ESO
│   ├── address-pool.yaml      # cluster IP pool for MetalLB
│   ├── argocd-install/        # Argo CD install manifests + ingress
│   └── root-app.yaml          # root Application watching apps/root
├── secrets/
│   ├── adguard.yaml           # adguard credentials from BitWarden
│   ├── <app>.yaml             # <app> credentials from BitWarden
│   └── ...
├── apps/
│   ├── root/
│   │   ├── kustomization.yaml # points to applicationset.yaml
│   │   └── applicationset.yaml# auto-discovers apps/*/*.yaml
│   └── <namespace>/           # e.g., dev/, default/, infra/, monitoring/
│       ├── <app-1>.yaml       # all-in-one manifest per app
│       ├── <app-2>.yaml       # ...
|       └──── ...
```
