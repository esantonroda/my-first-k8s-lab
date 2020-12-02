# upgrade.md

## References:

* [Upgrade an Azure Kubernetes Service (AKS) cluster](https://docs.microsoft.com/en-us/azure/aks/upgrade-cluster)

## Steps:

Check possible upgrade and perform it

```bash
$ az aks get-upgrades --resource-group test-aks --name aks-clu -o table
The behavior of this command has been altered by the following extension: aks-preview
Name     ResourceGroup    MasterVersion    Upgrades
-------  ---------------  ---------------  --------------
default  test-aks         1.18.10          1.19.1, 1.19.3

$ kubectl get no -o wide
NAME                                STATUS   ROLES   AGE   VERSION    INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
aks-nodepool1-38324427-vmss000000   Ready    agent   75m   v1.18.10   10.240.0.4    <none>        Ubuntu 18.04.5 LTS   5.4.0-1031-azure   docker://19.3.12

$ az aks upgrade --resource-group test-aks --name aks-clu --kubernetes-version 1.19.1
The behavior of this command has been altered by the following extension: aks-preview
Kubernetes may be unavailable during cluster upgrades.
 Are you sure you want to perform this operation? (y/N): y
Since control-plane-only argument is not specified, this will upgrade the control plane AND all nodepools to version 1.19.1. Continue? (y/N): y
{- Finished ..
...
}

```

Upgrade with care (testing), now you're upgraded to containerd

```bash

$ az aks get-upgrades --resource-group test-aks --name aks-clu -o table
The behavior of this command has been altered by the following extension: aks-preview
Name     ResourceGroup    MasterVersion    Upgrades
-------  ---------------  ---------------  ----------
default  test-aks         1.19.1           1.19.3

$ kubectl get no -o wide -A
NAME                                STATUS   ROLES   AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
aks-nodepool1-38324427-vmss000000   Ready    agent   93m   v1.19.1   10.240.0.4    <none>        Ubuntu 18.04.5 LTS   5.4.0-1031-azure   containerd://1.4.1+azure

```
