## Setup AdGuard Sync credentials
```bash
kubectl create secret generic adguardhome-password \ 
    --from-literal=password='your_adguardhome_password' \ 
    --from-literal=username='your_adguardhome_username' -n dns
```
