# create-aks-cluster-with-az-cli

## References:

* [Quickstart: Deploy an Azure Kubernetes Service cluster using the Azure CLI](https://docs.microsoft.com/es-es/azure/aks/kubernetes-walkthrough?source=docs)
* [Kubernetes on Microsoft Azure Kubernetes Service (AKS) with Autoscaling](https://zero-to-jupyterhub.readthedocs.io/en/latest/kubernetes/microsoft/step-zero-azure-autoscale.html)

## Create resource group

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

Testing

```bash
❯ az group create \
>             --name test-aks \
>             --location westeurope \
>             --output table
Location    Name
----------  --------
westeurope  test-aks
```

## Check privileges of your account

```
az provider show -n Microsoft.OperationsManagement -o table
az provider show -n Microsoft.OperationalInsights -o table
```

```bash
❯ az provider show -n Microsoft.OperationsManagement -o table
Namespace                       RegistrationPolicy    RegistrationState
------------------------------  --------------------  -------------------
Microsoft.OperationsManagement  RegistrationRequired  Registered
❯ az provider show -n Microsoft.OperationalInsights -o table
Namespace                      RegistrationPolicy    RegistrationState
-----------------------------  --------------------  -------------------
Microsoft.OperationalInsights  RegistrationRequired  Registered
```

They are not granted by default, you need to grant them

```
az provider register --namespace Microsoft.OperationsManagement
az provider register --namespace Microsoft.OperationalInsights
```

## Create the cluster

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

❯ az aks get-versions --location westeurope -o table
The behavior of this command has been altered by the following extension: aks-preview
KubernetesVersion    Upgrades
-------------------  -----------------------
1.20.5               None available
1.20.2               1.20.5
1.19.9               1.20.2, 1.20.5
1.19.7               1.19.9, 1.20.2, 1.20.5
1.18.17              1.19.7, 1.19.9
1.18.14              1.18.17, 1.19.7, 1.19.9


az aks create --name myAKSCluster \
              --resource-group test-aks \
              --node-count 2 \
              --node-vm-size Standard_D2s_v3 \
              --enable-vmss \
              --enable-cluster-autoscaler \
              --min-count 1 \
              --max-count 3 \
              --kubernetes-version 1.18.14 \
              --output table

## with autoscale

❯ az aks create --name myAKSCluster \
>               --resource-group test-aks \
>               --node-count 2 \
>               --node-vm-size Standard_D2s_v3 \
>               --enable-vmss \
>               --enable-cluster-autoscaler \
>               --min-count 1 \
>               --max-count 3 \
>               --kubernetes-version 1.17.13 \
>               --output table
The behavior of this command has been altered by the following extension: aks-preview

DnsPrefix                   EnablePodSecurityPolicy    EnableRbac    Fqdn                                                          KubernetesVersion    Location    MaxAgentPools    Name          NodeResourceGroup                    ProvisioningState    ResourceGroup
--------------------------  -------------------------  ------------  ------------------------------------------------------------  -------------------  ----------  ---------------  ------------  -----------------------------------  -------------------  ---------------
myAKSClust-test-aks-266215  False                      True          myaksclust-test-aks-266215-2276faf4.hcp.westeurope.azmk8s.io  1.17.13              westeurope  10               myAKSCluster  MC_test-aks_myAKSCluster_westeurope  Succeeded            test-aks



```
## Retrieve credentials for kubectl

```
az aks get-credentials --resource-group test-aks --name myAKSCluster
az aks get-credentials --resource-group test-aks --name myAKSCluster --admin

❯ az aks get-credentials --resource-group test-aks --name myAKSCluster --admin
The behavior of this command has been altered by the following extension: aks-preview
A different object named myAKSCluster already exists in your kubeconfig file.
Overwrite? (y/n): y
A different object named clusterAdmin_test-aks_myAKSCluster already exists in your kubeconfig file.
Overwrite? (y/n): y
Merged "myAKSCluster-admin" as current context in /home/dataniard/.kube/config

```
❯ kubectl get nodes -o wide -ALL
NAME                                STATUS   ROLES   AGE     VERSION    INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME   L
aks-nodepool1-19768975-vmss000000   Ready    agent   3m3s    v1.17.13   10.240.0.4    <none>        Ubuntu 16.04.7 LTS   4.15.0-1106-azure   docker://19.3.14
aks-nodepool1-19768975-vmss000001   Ready    agent   3m19s   v1.17.13   10.240.0.5    <none>        Ubuntu 16.04.7 LTS   4.15.0-1106-azure   docker://19.3.14

Scale nodes to zero
```
az aks scale --resource-group test-aks --name myAKSCluster --node-count 0 --nodepool-name nodepool1

❯ az aks scale --resource-group test-aks --name myAKSCluster --node-count 0 --nodepool-name nodepool1
The behavior of this command has been altered by the following extension: aks-preview
Cannot scale cluster autoscaler enabled node pool.
```
## Stop/start the cluster (aks-preview)
```
az aks stop --name myAKSCluster --resource-group test-aks

## Elapsed time (2m 46s)
❯ az aks stop --name myAKSCluster --resource-group test-aks
The behavior of this command has been altered by the following extension: aks-preview

az aks start --name myAKSCluster --resource-group test-aks

## Elapsed time (3m 8s)
❯ az aks start --name myAKSCluster --resource-group test-aks -o table
The behavior of this command has been altered by the following extension: aks-preview

```
## Delete your tests
```
az group delete --name test-aks --yes --no-wait
```

## Upgrading the cluster
First check 

```
❯ az aks get-upgrades --name myAKSCluster --resource-group test-aks -o table
The behavior of this command has been altered by the following extension: aks-preview
Name     ResourceGroup    MasterVersion    Upgrades
-------  ---------------  ---------------  -------------------------
default  test-aks         1.17.13          1.17.16, 1.18.10, 1.18.14

❯ az aks nodepool list --cluster-name myAKSCluster --resource-group test-aks -o table
The behavior of this command has been altered by the following extension: aks-preview
Name       OsType    VmSize           Count    MaxPods    ProvisioningState    Mode
---------  --------  ---------------  -------  ---------  -------------------  ------
nodepool1  Linux     Standard_D2s_v3  1        110        Succeeded            System
```

Starting the upgrade

```
az aks upgrade  --resource-group test-aks --name myAKSCluster --kubernetes-version 1.18.14 -o table

❯ az aks upgrade  --resource-group test-aks --name myAKSCluster --kubernetes-version 1.19.6 -o table
The behavior of this command has been altered by the following extension: aks-preview
Kubernetes may be unavailable during cluster upgrades.
 Are you sure you want to perform this operation? (y/N): y
Since control-plane-only argument is not specified, this will upgrade the control plane AND all nodepools to version 1.19.6. Continue? (y/N): y
DnsPrefix                   EnablePodSecurityPolicy    EnableRbac    Fqdn                                                          KubernetesVersion    Location    MaxAgentPools    Name          NodeResourceGroup                    ProvisioningState    ResourceGroup
--------------------------  -------------------------  ------------  ------------------------------------------------------------  -------------------  ----------  ---------------  ------------  -----------------------------------  -------------------  ---------------
myAKSClust-test-aks-266215  False                      True          myaksclust-test-aks-266215-2276faf4.hcp.westeurope.azmk8s.io  1.19.6               westeurope  10               myAKSCluster  MC_test-aks_myAKSCluster_westeurope  Succeeded            test-aks
```

During the upgrade we can check the effects on the cluster.

```
❯ kubectl get no,po -ALL -o wide
NAME                                     STATUS                     ROLES   AGE    VERSION    INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME          L
node/aks-nodepool1-19768975-vmss000002   Ready,SchedulingDisabled   agent   22m    v1.18.14   10.240.0.4    <none>        Ubuntu 18.04.5 LTS   5.4.0-1039-azure   docker://19.3.14
node/aks-nodepool1-19768975-vmss000004   Ready                      agent   2m6s   v1.19.6    10.240.0.5    <none>        Ubuntu 18.04.5 LTS   5.4.0-1039-azure   containerd://1.4.3+azure

NAMESPACE     NAME                                            READY   STATUS    RESTARTS   AGE     IP           NODE                                NOMINATED NODE   READINESS GATES   L
kube-system   pod/azure-ip-masq-agent-2tvvl                   1/1     Running   0          4m22s   10.240.0.4   aks-nodepool1-19768975-vmss000002   <none>           <none>
kube-system   pod/azure-ip-masq-agent-rfjk4                   1/1     Running   0          2m6s    10.240.0.5   aks-nodepool1-19768975-vmss000004   <none>           <none>
kube-system   pod/coredns-79766dfd68-gjck7                    1/1     Running   0          72s     10.244.1.8   aks-nodepool1-19768975-vmss000004   <none>           <none>
kube-system   pod/coredns-79766dfd68-zfnr2                    1/1     Running   0          87s     10.244.1.7   aks-nodepool1-19768975-vmss000004   <none>           <none>
kube-system   pod/coredns-autoscaler-66c578cddb-l72gg         1/1     Running   0          87s     10.244.1.3   aks-nodepool1-19768975-vmss000004   <none>           <none>
kube-system   pod/dashboard-metrics-scraper-6f5fb5c4f-95r9f   1/1     Running   0          87s     10.244.1.5   aks-nodepool1-19768975-vmss000004   <none>           <none>
kube-system   pod/kube-proxy-dfbp8                            1/1     Running   0          3m35s   10.240.0.4   aks-nodepool1-19768975-vmss000002   <none>           <none>
kube-system   pod/kube-proxy-qqsjj                            1/1     Running   0          2m6s    10.240.0.5   aks-nodepool1-19768975-vmss000004   <none>           <none>
kube-system   pod/kubernetes-dashboard-56dbcd8bf5-788r9       1/1     Running   0          87s     10.244.1.2   aks-nodepool1-19768975-vmss000004   <none>           <none>
kube-system   pod/metrics-server-7f5b4f6d8c-hk88l             1/1     Running   0          87s     10.244.1.4   aks-nodepool1-19768975-vmss000004   <none>           <none>
kube-system   pod/tunnelfront-766fbb7fc6-dhxrj                1/1     Running   0          87s     10.244.1.6   aks-nodepool1-19768975-vmss000004   <none>           <none>

❯ kubectl get no,po -ALL -o wide
NAME                                     STATUS   ROLES   AGE   VERSION    INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME   L
node/aks-nodepool1-19768975-vmss000002   Ready    agent   13m   v1.18.14   10.240.0.4    <none>        Ubuntu 18.04.5 LTS   5.4.0-1039-azure   docker://19.3.14

NAMESPACE     NAME                                            READY   STATUS    RESTARTS   AGE     IP           NODE                                NOMINATED NODE   READINESS GATES   L
kube-system   pod/coredns-79766dfd68-8tlhr                    1/1     Running   0          31s     10.244.0.8   aks-nodepool1-19768975-vmss000002   <none>           <none>
kube-system   pod/coredns-79766dfd68-lbldj                    1/1     Running   0          46s     10.244.0.5   aks-nodepool1-19768975-vmss000002   <none>           <none>
kube-system   pod/coredns-autoscaler-66c578cddb-jzg4t         1/1     Running   0          46s     10.244.0.6   aks-nodepool1-19768975-vmss000002   <none>           <none>
kube-system   pod/dashboard-metrics-scraper-6f5fb5c4f-nkwds   1/1     Running   0          46s     10.244.0.7   aks-nodepool1-19768975-vmss000002   <none>           <none>
kube-system   pod/kube-proxy-s6xrs                            1/1     Running   0          4m34s   10.240.0.5   aks-nodepool1-19768975-vmss000003   <none>           <none>
kube-system   pod/kube-proxy-tkxl5                            1/1     Running   0          5m37s   10.240.0.4   aks-nodepool1-19768975-vmss000002   <none>           <none>
kube-system   pod/kubernetes-dashboard-56dbcd8bf5-lcslb       1/1     Running   0          46s     10.244.0.4   aks-nodepool1-19768975-vmss000002   <none>           <none>
kube-system   pod/metrics-server-7f5b4f6d8c-4sd4z             1/1     Running   0          46s     10.244.0.2   aks-nodepool1-19768975-vmss000002   <none>           <none>
kube-system   pod/tunnelfront-766fbb7fc6-2xhbt                1/1     Running   0          46s     10.244.0.3   aks-nodepool1-19768975-vmss000002   <none>           <none>
```

