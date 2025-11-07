# Compute Resource

## Request and Limits
- `LimitRange`: limits resource usage per container / pod in a namespace
  - `type`: whether it applies to pod or containers
  - `defaultRequest`: default resource the application will request
  - `default`: max resource the application can use
- `Quota`: limits total resources available in a namespace
  - if a namespace has quota set, applications must have resource settings at `pod.spec.containers.resources`
- Difference:
  - `LimitRange`: set default restrictions on each application
  - `Quota`: define max resources that can be used in a namespace by all applications

## Demo: Set Quota on Namespace

```sh
kubectl create ns limited
kubectl create quota qtest --hard=pods=3,cpu=100m,memory=500Mi -n limited
# check quota applied
kubectl describe ns limited
kubectl create deploy nginx --image=nginx --replicas=3 -n limited
# pods are not scheduled due to failed quota, must specify cpu and memory
kubectl get all -n limited
# set resource request and limit at deployment
kubectl set resources deploy nginx --requests=cpu=100m,memory=5Mi --limits=cpu=200m,memory=20Mi -n limited
# only one pod is scheduled, rest is out of cpu quota
kubectl get all -n limited
# raise quota by edit
kubectl edit quota -n limited
```

## Demo: Set LimitRange

https://kubernetes.io/docs/concepts/policy/limit-range/

```sh
kubectl create ns limitrange
kubectl apply -f limitrange.yaml -n limitrange
kubectl run limitpod -n limitrange --image=nginx
# check the default is applied to pod
kubectl describe pod limitpod -n limitrange
```

`limitrange.yaml`
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```
