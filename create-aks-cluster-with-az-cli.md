# create-aks-cluster-with-az-cli

References:

* [Quickstart: Deploy an Azure Kubernetes Service cluster using the Azure CLI](https://docs.microsoft.com/es-es/azure/aks/kubernetes-walkthrough?source=docs)
* [Kubernetes on Microsoft Azure Kubernetes Service (AKS) with Autoscaling](https://zero-to-jupyterhub.readthedocs.io/en/latest/kubernetes/microsoft/step-zero-azure-autoscale.html)

Create resource group

```
az group create \
              --name=<RESOURCE-GROUP-NAME> \
              --location=<LOCATION> \
              --output table

az group create \
            --name test-aks \
            --location westeurope \
            --output table

```

Check privileges of your account

```
az provider show -n Microsoft.OperationsManagement -o table
az provider show -n Microsoft.OperationalInsights -o table
```

They are not granted by default, you need to grant them

```
az provider register --namespace Microsoft.OperationsManagement
az provider register --namespace Microsoft.OperationalInsights
```

Create the cluster

```bash
-- one node
az aks create \
        --resource-group test-aks \
        --name aks-clu \
        --node-count 1 \
        --enable-addons monitoring \
        --generate-ssh-keys

-- with autoscaling

az network vnet create \
    --resource-group <RESOURCE-GROUP-NAME> \
    --name <VNET-NAME> \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name <SUBNET-NAME> \
    --subnet-prefix 10.240.0.0/16

VNET_ID=$(az network vnet show \
    --resource-group <RESOURCE-GROUP-NAME> \
    --name <VNET-NAME> \
    --query id \
    --output tsv)
SUBNET_ID=$(az network vnet subnet show \
    --resource-group <RESOURCE-GROUP-NAME> \
    --vnet-name <VNET-NAME> \
    --name <SUBNET-NAME> \
    --query id \
    --output tsv)

SP_PASSWD=$(az ad sp create-for-rbac \
    --name <SERVICE-PRINCIPAL-NAME> \
    --role Contributor \
    --scope $VNET_ID \
    --query password \
    --output tsv)
SP_ID=$(az ad sp show \
    --id http://<SERVICE-PRINCIPAL-NAME> \
    --query appId \
    --output tsv)

az aks create --name <CLUSTER-NAME> \
              --resource-group <RESOURCE-GROUP-NAME> \
              --ssh-key-value ssh-key-<CLUSTER-NAME>.pub \
              --node-count 3 \
              --node-vm-size Standard_D2s_v3 \
              --enable-vmss \
              --enable-cluster-autoscaler \
              --min-count 3 \
              --max-count 6 \
              --kubernetes-version 1.12.7 \
              --service-principal $SP_ID \
              --client-secret $SP_PASSWD \
              --dns-service-ip 10.0.0.10 \
              --docker-bridge-address 172.17.0.1/16 \
              --network-plugin azure \
              --network-policy azure \
              --service-cidr 10.0.0.0/16 \
              --vnet-subnet-id $SUBNET_ID \
              --output table

```
Retrieve credentials for kubectl

```
az aks get-credentials --resource-group test-aks --name aks-clu
```
Delete ypur tests

```
az group delete --name test-aks --yes --no-wait
```