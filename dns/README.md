## Setup AdGuard Sync credentials
```bash
kubectl create secret generic adguardhome-password \ 
    --from-literal=password='your_adguardhome_password' \ 
    --from-literal=username='your_adguardhome_username' -n dns
```

## Add AdGuardHome to CoreDNS configmap fallback:
1. Edit the CoreDNS configmap:
```bash
kubectl edit configmap coredns -n kube-system
```
2. Replace the `forward` line with the following:
```
    forward . <ADGUARDHOME_IP> <ADGUARDHOME_IP_2>
```
This will use AdGuardHome as the primary DNS server and a secondary one as a fallback, instead of using the default Kubernetes CoreDNS server.

You may also use `/etc/resolv.conf` to forward to the node's own DNS resolver, but it depends on whether it's well configured or not. *Since it's Linux, we never know.*

Ideally, since DNS is required for fetching the container image, you would have AdGuardHome as first and then a public DNS server as second (fallback).