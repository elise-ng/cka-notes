# Scaling Applications

## Manually Scale

- `kubectl scale deployment myapp --replicas=3`

## HorizontalPodAutoscaler

- `kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=80`
- `kubectl get hpa`
- Watches cpu usage and scale automatically to the target average (e.g. 80% in example above)
- Default is 80% cpu usage
- Default scale down if below threshold for 5 minutes
  - per HPA config: `spec.behavior.scaleDown.stabilizationWindowSeconds`)
  - cluster-wide config: kube-controller-manager manifest, start up flag `--horizontal-pod-autoscaler-downscale-delay=180s`
- Value is relative to the cpu resource request
- Metric Server is required for HPA to work

### HPA Demo

Check latest metric server releases at https://github.com/kubernetes-sigs/metrics-server/releases

```sh
# install metric server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.8.0/components.yaml
# set --kubelet-insecure-tls=true in startup flag
kubectl -n kube-system edit deploy metrics-server
# verify install
kubectl top pod -A
# demo web app
kubectl create deploy webstress --image=nginx
kubectl autoscale deployment webstress --min=2 --max=4 --cpu-percent=80
# we will see 2 pods right now
kubectl get deploy webstress
# edit scale down timing at spec.behavior.scaleDown.stabilizationWindowSeconds
kubectl edit hpa webstress
```
