## Creating gitea-runner secrets

```bash
kubectl create secret generic gitea-runner-token \
    --from-literal=REGISTRATION_TOKEN='your_gitea_runner_token' -n dev
```