# Dapr Setup

```bash

flux create source helm redis \
    --url=https://charts.bitnami.com/bitnami \
    --interval=10m \
    --export > redis-source.yaml

flux create helmrelease redis \
    --interval=10m \
    --source=HelmRepository/redis \
    --chart=redis \
    --chart-version=12.1.1 \
    --release-name=redis \
    --target-namespace=redis-system \
    --export > redis-release.yaml

```
