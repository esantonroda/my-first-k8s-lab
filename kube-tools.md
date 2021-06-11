# kube-tools

## install kubectl in ubuntu

- [Install and Set Up kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

## kube-ps1

for bash shell, clone the [repo](https://github.com/jonmosco/kube-ps1) and source kube-ps1.sh from .bashrc

```bash
source /path/to/kube-ps1.sh
PS1='[\u@\h \W $(kube_ps1)]\$ '
```

## use my dotfiles

If you want to use zsh, you can use my [dotfiles](https://github.com/esantonroda/dotfiles), based on:

- [powerlevel10k](https://github.com/romkatv/powerlevel10k)
- [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)
- kubectl+kubens+kubectx
- completion, syntax-highligthing and autosuggestions
- config adapted to aws, azure, k8s ...
- works on several distros, even web consoles
- Maybe won't be perfect but it's mine

## References

- [Install and Set Up kubectl](https://v1-18.docs.kubernetes.io/docs/tasks/tools/install-kubectl/)
- [kube-ps1: Kubernetes prompt for bash and zsh](https://github.com/jonmosco/kube-ps1)
- [powerlevel10k](https://github.com/romkatv/powerlevel10k)
- [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)
- [kube-ps1: Kubernetes prompt for bash and zsh](https://github.com/jonmosco/kube-ps1)
