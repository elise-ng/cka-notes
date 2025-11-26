# Storage
- Pod = Containers + Volumes
- PVC -> Specify StorageClass -> Provisioner bounds PV to PVC
- Pod Volumes: part of Pod spec, storage reference hard coded
  - types e.g. emptyDir, configMap
  - emptyDir: short-lived temp storage, useful for containers ipc
- Persistent Volumes: api resource that represent storage
  - PV can be created manually or automatically with StorageClass + provisioner
  - `storageClassName`: selector label to allow PVC binding
  - `capacity`: size in GiB
  - `accessMode`: RW or Read Only
  - specific type of stroage e.g. SSD or HDD
- PersistentVolumeClaim: api resource that allows pod to connect to any type of storage provided at the environment
  - make the application agnostic of the storage type this is provided
  - flexible for the application to pick up any storage available at the deployment env
- PersistentVolumeReclaimPolicy: set on PV to determine what happends when it is no longer bound to a PVC
  - default: `Retain`
  - `Delete`: pv will be deleted once unbound
  - `Recycle`: pv will recycle back to pool of unused PVs, can be reused by another PVC that matches label
- ConfigMap: api resource used to store site-specific data
  - stores configs in etcd database
  - env var, startup params, config files
  - mounted as volume by pods
  - `kubectl set env`: to update the env var in the config map
  - `kubectl create cm mydir --from-file=/my/directory/`: config map can be multiple files
  - `pod.spec.containers.volumeMounts.subPath`: only mount part of config map
- Secret: base64 encoded ConfigMap
 
## Example
`pv.yaml`
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  storageClassName: standard
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```
- hostPath: provided by the node running the pod, not very flexible / cloud native

`pvc.yaml`
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

`pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: my-storage
      mountPath: /usr/share/nginx/html
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

## Demo: ConfigMap
```sh
kubectl create cm webindex --from-file=index.html
kubectl describe cm webindex
kubectl create deploy webserver --image=nginx
kubectl edit deploy webserver
# add volume and volume mount in template, see below yaml
kubectl exec -it webserver-xxx -- ls /usr/local/nginx/html
```
```yaml
# sepc.template.spec
volumes:
- name: cm-volume
  configMap:
    name: webindex
# sepc.template.containers[0]
volumeMounts:
- mountPath: /usr/share/nginx/html
  name: cm-volume
```
