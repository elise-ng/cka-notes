# Deploy Applications

## Deployment

- starting pods in a scalable way
- uses ReplicaSet to manage scalability
- allows zero-downtime application updates

### Example Deploy

```
# create
kubectl create deploy mydeploy --image=nginx:1.28 --replicas=3
# scale down
kubectl scale deployment mydeploy --replicas=4
# apply image rolling update
kubectl set image deploy mydeploy nginx=nginx:1.29
```

## DaemonSet

- starts one application instance on each node
- commonly used to start agents
- toleration needed to allow pods to run on control plane nodes (override taint)

### Example DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: daemon
  name: daemon
spec:
  selector:
    matchLabels:
      app: daemon
  template:
    metadata:
      labels:
        app: daemon
    spec:
      containers:
      - image: nginx
        name: nginx
```

## StatefulSets

- Stateless: does not store session data, easy to load balance: traffic can be served by any pods
- Stateful: saves session data to persistent storage, e.g. database
- StatefulSet:
  - guarantees ordering and uniqueness of pods
  - maintains sticky identifier for each pod it creates
  - pods in StatefulSet not interchangeable, each pod has a persistent identifier maintained during reschedule
  - identifiers make it easy to match storage / networking
  - ordered deployment. scaling, rolling update
- Just use deployment if these features are not required
- Persistent Volume needs to be provisioned by a storage provisioner
- Headless service needed for networking
- Pods not guaranteed to be stopped while deleting StatefulSet, best to scale down to zero before delete

### Example StatefulSet

Ref: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#components

## Running Pods directly

- No workload protection (does not recover once down)
- No load balancing
- No zero-downtime application update
- Only use for testing or trouble shooting

## Init Containers

- Under Pod spec
- Run preparation before running main container

### Exmaple InitContainer

Ref: https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use
