# Learn FluxCD v2.x - Bootstrap

The bootstrapping repo for a FluxCD v2.x implementation

## Kubernetes Cluster Setup

### Setup Shell Variables

These variables will be used throughout the demo.

```bash
export NAME=cdw-kubernetes-20210302
export LOCATION=westus2
export ACR_NAME=cdwms
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
--network-plugin azure \
--generate-ssh-keys \
--enable-managed-identity \
--enable-pod-identity \
--enable-addons monitoring \
--attach-acr $ACR_NAME

# Get the credentials
az aks get-credentials -n $NAME -g $NAME --overwrite
```

## Push the Sample Image to Your ACR

```bash
docker pull ghcr.io/stefanprodan/podinfo:5.0.0
docker tag ghcr.io/stefanprodan/podinfo:5.0.0 $ACR_NAME.azurecr.io/podinfo:5.0.0
az acr login -n $ACR_NAME
docker push $ACR_NAME.azurecr.io/podinfo:5.0.0
```

## Setup an AAD Pod Identity

```bash
# Create an identity in Azure and assign it a role to pull from ACR  (note: the identity's resourceGroup should match the desired ACR):
az identity create -n acr-sync -g $NAME
az role assignment create --role AcrPull --assignee-object-id "$(az identity show -n acr-sync -g $NAME -o tsv --query principalId)"

# Fetch the clientID and resourceID to configure the AzureIdentity spec below:
az identity show -n acr-sync -g $NAME -otsv --query clientId
az identity show -n acr-sync -g $NAME -otsv --query id

# Use the above values to update the clusters/my-cluster/podinfo/flux/config-patches.yaml file
```

## Add Flux Bootstrap

Here, we will use the instructions from [this page](https://toolkit.fluxcd.io/get-started/#install-flux-components) to add the FluxCD 2.x bootstrap components to the repo.

```bash
# If necessary, install FluxCD CLI
curl -s https://toolkit.fluxcd.io/install.sh | sudo bash

# Set the creds for your GitHub repo
export GITHUB_USER=<your-username>
export GITHUB_TOKEN=<your-token>

# Confirm that everything is good to go
flux check --pre

# Setup the repo and install on the cluster
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=learn-flux-bootstrap \
  --branch=main \
  --path=./clusters/my-cluster \
  --namespace=flux-system \
  --components-extra=image-reflector-controller,image-automation-controller \
  --personal \
  --verbose

# Test service with Port Forwarding
kubectl -n demo1 port-forward service/podinfo 9898:9898

# Test the container tag scanning
flux get image repository podinfo -n demo1
```

## Setup an ACR Pull Secret

```bash
kubectl create secret docker-registry acr-cdwms-creds \
    --namespace demo1 \
    --docker-server=$ACR_NAME.azurecr.io \
    --docker-username=$ACR_NAME \
    --docker-password=<<YOUR ACR PASSWORD >>
```

## Cluster Clean Up

```bash
az group delete --name $NAME
```
