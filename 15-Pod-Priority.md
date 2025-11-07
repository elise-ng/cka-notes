# Pod Priority

- kube-schedule does not have priority by default
- use `PriorotyClass` to determine order of pod schedule and evict when there are resource constraint
- higher value = high priority
- assign `priorityClassName` to each pod
- we can also set `globalDefault` for pods with no specific priority class set
- when PriorityClass is used and cluster is out of resources, lower priority pods will be evicted to make space
- `preemptionPolicy` can be set to never to ensure pods will never be evicted

## Demo

```sh
kubectl create priorityclass high-priority --value=1000 --description="high priority" --preemption-policy="Never"
kubectl create priorityclass mid-priority --value=125 --description="mid priority" --global-default=true
kubectl run testpod --image=nginx
# check mid-priority is used as default
kubectl get pods testpod -o yaml | grep -B2 -i priorityClass
# try high priority
kubectl create deploy highprio --image=nginx
# manually set `priorityClassName: high-priority` in pod spec
kubectl edit deploy highprio
# it is now applied
kubectl get pod highprio-56588c7c77-v4q7q -o yaml | grep -B2 -i priorityClass
```
