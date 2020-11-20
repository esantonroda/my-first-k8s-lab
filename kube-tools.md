# kube-tools

## References

* [Install and Set Up kubectl](https://v1-18.docs.kubernetes.io/docs/tasks/tools/install-kubectl/)
* [kube-ps1: Kubernetes prompt for bash and zsh](https://github.com/jonmosco/kube-ps1)


## install kubectl in ubuntu

```
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

## kube-ps1

for bash shell, clone the repo and source kube-ps1.sh from .bashrc

```
source /path/to/kube-ps1.sh
PS1='[\u@\h \W $(kube_ps1)]\$ '
```
