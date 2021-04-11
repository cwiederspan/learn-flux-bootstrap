# Dapr Setup

```bash

flux create source helm dapr \
    --url=https://dapr.github.io/helm-charts \
    --interval=10m \
    --export > dapr-source.yaml

flux create helmrelease dapr \
    --interval=10m \
    --source=HelmRepository/dapr \
    --chart=dapr \
    --release-name=dapr \
    --target-namespace=dapr-system \
    --export > dapr-release.yaml

```
