# create-aks-cluster-with-az-cli

References:

* [Quickstart: Deploy an Azure Kubernetes Service cluster using the Azure CLI](https://docs.microsoft.com/es-es/azure/aks/kubernetes-walkthrough?source=docs)


Create resource group

```
az group create --name myResourceGroup --location eastus
```

Check privileges of your account

```bash
az provider show -n Microsoft.OperationsManagement -o table
az provider show -n Microsoft.OperationalInsights -o table
```