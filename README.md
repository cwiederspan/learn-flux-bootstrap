# Learn FluxCD v2.x - Bootstrap

The bootstrapping repo for a FluxCD v2.x implementation

## Kubernetes Cluster Setup

### Setup Shell Variables

These variables will be used throughout the demo.

```bash
export NAME=cdw-kubernetes-20210302
export LOCATION=westus2
```

### Create an Azure Resource Group

```bash
az group create -n $NAME -l $LOCATION
```

### Create an Azure Kubernetes Cluster

```bash
# Check available versions per location
az aks get-versions -l $LOCATION -o table

# Create cluster
az aks create \
--resource-group $NAME \
--name $NAME \
--location $LOCATION \
--kubernetes-version 1.19.7 \
--node-count 2 \
--generate-ssh-keys \
--enable-managed-identity \
--enable-addons monitoring

# Get the credentials
az aks get-credentials -n $NAME -g $NAME --overwrite
```

## Add Flux Bootstrap

Here, we will use the instructions from [this page](https://toolkit.fluxcd.io/get-started/#install-flux-components) to add the FluxCD 2.x bootstrap components to the repo.

```bash
curl -s https://toolkit.fluxcd.io/install.sh | sudo bash

export GITHUB_USER=<your-username>
export GITHUB_TOKEN=<your-token>

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=learn-flux-bootstrap \
  --branch=main \
  --path=./clusters/my-cluster \
  --namespace=flux-system \
  --personal \
  --verbose
```
