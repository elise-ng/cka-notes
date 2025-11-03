# k8s Metric Server

Not included in default install.

https://github.com/kubernetes-sigs/metrics-server

https://github.com/kubernetes-sigs/metrics-server/releases

```sh
# apply manifest (look up github for the latest release supporting cluster k8s version)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.8.0/components.yaml

# it will not support insecure tls by default
kubectl edit -n kube-system deployment metrics-server
# then, add --kubelet-insecure-tls flag to container args

# it will reconcile after a few minutes
kubectl top pods -A
```
