## quick scale deployment accross namespaces
```bash
k get deploy | awk 'NR > 1 {print $1}' | xargs -n1  kubectl scale deploy --replicas=0
```