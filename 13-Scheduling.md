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

## Taints and Tolerations

https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

- taints: are applied to node to mark that the node should not accept any pod that doesn't tolerate the taint
- tolerations: are applied to pods and allow (not require) pods to schedule on nodes with matching taints
- vs affinity: affinity used on pods to atract them to specific nodes; taints used on nodes to repel set of pods
- ensure dedicated nodes are used for dedicated tasks
- effects:
  - `NoSchedule`: does not schedule new pods
  - `PreferNoSchedule`: does not schedule new pods unless there is not other option
  - `NoExecute`: migrates existing pods away from this node
- control plane nodes automatically get taints to avoid scheduling user pods
- `kubectl drain` and `kubectl cordon` applies taints
- taints are set automatically when node has critical condition e.g. out of disk space, network unavailable, memory pressure
- `kubectl taint` or `kubectl edit` to manually taint nodes
- when defining toleration, the pod needs `key`, `operator`, `value`, `effect`
  - default operator is `Equal` but `Exists` is also commonly used

## Demo: Taint

```sh
kubectl taint node worker-2 diskType=hdd:NoSchedule
kubectl create deploy testtaint --image=nginx --replicas=6
# check all pods are on worker-1 now:
kubectl get pods -o wide --sort-by="{.spec.nodeName}"
# add toleration
kubectl edit deploy testtaint
# check pods are now running on worker-2 too
kubectl get pods -o wide --sort-by="{.spec.nodeName}"
# untaint node
kubectl taint node worker-2 diskType=hdd:NoSchedule-
```

```yaml
template:
  spec:
    tolerations:
    - effect: NoSchedule
      key: diskType
      operator: Equal
      value: hdd
```
