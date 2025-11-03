## Static pod at /etc/kubernetes/manifests

(Pearson CKA Course Lab 3)

Kubelet will run the pod automatically. Used for kube internal core service, or agent pods.

Advantage: Guarantee accessibility even if API server (control plane) is down.

Can be used during recovery scenrio when API server is down as well.

```sh
# generate template
# kubectl run POD_NAME --image=IMAGE_NAME --dry-run=client -o yaml > POD_NAME.yaml
kubectl run staticpod --image=nginx --dry-run=client -o yaml > staticpod.yaml

sudo mv staticpod.yaml /etc/kubernetes/manifests

# it should be picked up as soon as it is placed at the dir
kubectl get pods
```

File Sample:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: staticpod
  name: staticpod
spec:
  containers:
  - image: nginx
    name: staticpod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
