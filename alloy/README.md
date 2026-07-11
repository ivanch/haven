# Helm configuration for Alloy (OSS monitoring collector)

This directory contains Helm configuration for deploying Grafana Alloy, an open-source observability data collector.

## Quick Install

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install alloy grafana/alloy --namespace alloy -f values.yaml
```
