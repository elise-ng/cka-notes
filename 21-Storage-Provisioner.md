# Storage Provisioner
- StorageClass: api resource that allows storage to be automatically provisioned
  - can also be used as property that connects PVC and PV without using an actual StorageClass resource
  - multiple StorageClass can co-exist in same cluster to provide different types of stroage
  - we can set one StorageClass as default `metadata.annotation."storageclass.kubernetes.io/is-default-class": true`
  - if no default is set and no storageClass property is set, pvc will be stuck at pending
- StorageProvisioner
  - external applications provided by k8s ecosystem / cloud vendor
  - automatically provisions PV for PVC

## Demo: Setting up NFS Storage Provisioner
```sh
# on control node
sudo apt install nfs-server -y
# on other nodes
sudo apt install nfs-client -y
# on control node
sudo mkdir /nfsexport
sudo sh -c 'echo "/nfsexport *(rw,no_root_squash)" > /etc/exports'
sudo systemctl restart nfs-server
sudo systemctl status nfs-server
# on other nodes
showmount -e kube-control-1
# on operator machine
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=kube-control-1 \
    --set nfs.path=/nfsexport
kubectl get pods
# the provisioner pod show be running
kubectl get storageclass
# note new storageclass "nfs-client"
kubectl get pvc,pv
kubectl apply -f nfs-test-pvc.yaml
kubectl get pvc,pv
```
`nfs-test-pvc.yaml`
```sh
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-nfs-pvc
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
```
