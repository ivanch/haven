
## Install nfs-subdir-external-provisioner
```bash
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=192.168.15.61 \
    --set nfs.path=/export/config \
    --set storageClass.name=nfs-client \
    --set storageClass.pathPattern='${.PVC.namespace}/${.PVC.annotations.nfs.io/storage-path}'
```
Make it default by:
```bash
current_default=$(kubectl get storageclass -o jsonpath='{.items[?(@.metadata.annotations.storageclass\.kubernetes\.io/is-default-class=="true")].metadata.name}')

if [ -n "$current_default" ]; then
  kubectl annotate storageclass "$current_default" storageclass.kubernetes.io/is-default-class- --overwrite
fi

kubectl annotate storageclass nfs-client storageclass.kubernetes.io/is-default-class=true --overwrite
```

PVC Usage:
```yaml
apiVersion: storage.k8s.io/v1
kind: PersistentVolumeClaim
metadata:
  name: app-config
  namespace: default
  annotations:
    nfs.io/storage-path: "app-config"
spec:
  storageClassName: "nfs-client"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

## Install MetalLB
```bash
kubectl create ns metallb-system
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb --namespace metallb-system
```

Configure MetalLB with the config map from [metallb-system/address-pool.yaml](metallb-system/address-pool.yaml), and apply it:
```bash
kubectl apply -f metallb-system/address-pool.yaml
```

## Install cert-manager
```bash
kubectl create namespace cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.1/cert-manager.yaml

# Create the private key for local CA
openssl genrsa -out ca.key 4096

# Create the root certificate, valid for 10 years
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt -subj "/CN=Homelab CA"

# Create secret and ClusterIssuer
kubectl create secret tls internal-ca-secret -cert=ca.crt --key=ca.key -n cert-manager
kubectl apply -f cert-manager/cluster-issuer.yaml
```