# Kubectl productivity

**Kubectl alias:**

Some of the 800 generated aliases are:

```bash
alias k='kubectl'
alias kg='kubectl get'
alias kgpo='kubectl get pod'

alias ksysgpo='kubectl --namespace=kube-system get pod'

alias krm='kubectl delete'
alias krmf='kubectl delete -f'
alias krming='kubectl delete ingress'
alias krmingl='kubectl delete ingress -l'
alias krmingall='kubectl delete ingress --all-namespaces'

alias kgsvcoyaml='kubectl get service -o=yaml'
alias kgsvcwn='watch kubectl get service --namespace'
alias kgsvcslwn='watch kubectl get service --show-labels --namespace'
```

**Install**: 

```bash
wget https://raw.githubusercontent.com/ahmetb/kubectl-aliases/master/.kubectl_aliases
echo "source ~/.kubectl_aliases" >> .zshrc
```

  
**\# get all pods sort by name**

**`kubectl get po -o jsonpath='{range.items[*]}{.metadata.name}{"\n"}{end}'`**

**\# Get all pods, which as restart\_count &gt; 0**

**`kubectl get po -o jsonpath='{range.items[?(@.status.containerStatuses[0].restartCount>0)]}{.status.containerStatuses[0].name}{"\n"}{end}'`  
  
\# Get all non-running pods  
`kubectl get po -o jsonpath='{range.items[?(@.status.phase != "Running")]}{.metadata.name}{"\n"}{end}'`  
  
\# Get most used cpu pods  
`kubectl top pods  | tail -n +2 |  sort -nr -k 2 | awk '{print $1}' | head -n 1`  
  
\# Get all nodes, and their IP  
`kubectl get no -o jsonpath='{range.items[*]}{.metadata.name}{"\t"}{.status.addresses[?(@.type=="InternalIP")].address}{"\n"}{end}'`**

