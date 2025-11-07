# Scheduling

## kube-scheduler
- handles finding a node to schedule pods
- nodes are filtered by:
  - resource requirements
  - affinity, anti-affinity
  - taints, tolerations
  - etc.
- it finds feasible nodes and pickes node with highest score
- `kubectl create` -> api server -> etcd -> scheduler -> kubelet -> cri

## Node Preferences
- `pod.spec.nodeSelector`: key-value pair to match node labels (e.g. `nodeSelector: diskType: ssd`)
- `pod.spec.nodeName`: specific node name (not recommended since workload is limited to a single defined node)

## Affinity and Anti-Affinity

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity

- node affinity: only schedule to node with matching label
- inter-pod affinity: only schedule to nodes running **pods** with matching label
- anti-affinity: can only be applied between pods -> make sure certains pods are never run on same node
- `requiredDuringSchedulingIgnoredDuringExecution`: requires the node to meet constraint
- `preferredDuringSchedulingIgnoredDuringExecution`: soft affinity that can be ignored if not possible to fulfill; weight can be assigned for priority
- at the moment affinity is only applied while scheduling pods and cannot affect running pods
- `matchExpression` used to define a `key`, `operator` and one or more `values` (instead of simple kv label matching)
  - operators: `In`, `Exists`, `Gt`, `Lt`
  - (anti-affinity) operators: `NotIn`, `DoesNotExist`
