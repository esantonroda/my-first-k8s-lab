# create-aks-cluster-with-az-cli

References:

* [Quickstart: Deploy an Azure Kubernetes Service cluster using the Azure CLI](https://docs.microsoft.com/es-es/azure/aks/kubernetes-walkthrough?source=docs)


Create resource group

```
az group create --name test-aks --location westeurope
```

Check privileges of your account

```
az provider show -n Microsoft.OperationsManagement -o table
az provider show -n Microsoft.OperationalInsights -o table
```

They are not granted by default you need to grant them

```
az provider register --namespace Microsoft.OperationsManagement
az provider register --namespace Microsoft.OperationalInsights
```

Create the cluster

```
az aks create --resource-group test-aks --name aks-clu --node-count 1 --enable-addons monitoring --generate-ssh-keys
```
Retrieve credentials for kubectl

```
az aks get-credentials --resource-group test-aks --name aks-clu
```
Scale nodes to zero
```
az aks scale --resource-group test-aks --name aks-clu --node-count 0 --nodepool-name agentpool
```
Delete your tests
```
az group delete --name test-aks --yes --no-wait
```
