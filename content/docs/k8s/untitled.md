# Linkerd Cheatsheet

### Uninject

Remove the Linkerd proxy from a Kubernetes config.

You can uninject resources contained in a single file, inside a folder and its sub-folders, or coming from stdin.

### Examples <a id="examples"></a>

```bash
# Uninject all the deployments in the default namespace.
kubectl get deploy -o yaml | linkerd uninject - | kubectl apply -f -

# Download a resource and uninject it through stdin.
curl http://url.to/yml | linkerd uninject - | kubectl apply -f -

# Uninject all the resources inside a folder and its sub-folders.
linkerd uninject <folder> | kubectl apply -f -
```



