# Etcd Backup / Restore

## Setup Dummy Service
To illustrate backup works

```sh
kubectl create deploy before --image-nginx --replicas=3
kubectl get deploy
kubectl get pods
```

## Backup

```sh
# cli tool is not installed by default
sudo apt install etcd-client

# check version
sudo etcdctl --help
# it should show API version 3, otherwise will need flag ETCDCTL_API=3 for commands

# get param for the running etcd process
ps aux | grep etcd
# look for the port and cert paths

# test connection
sudo etcdctl --endpoints=localhost:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get / --prefix --keys-only

# take snapshot
sudo etcdctl --endpoints=localhost:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key snapshot save /tmp/etcd-backup.db

# check summary of dump file
sudo etcdctl --write-out=table snapshot status /tmp/etcd-backup.db
```

## Restore

- We will need to stop core k8s services before restoring the dump

```sh
# Stop core services defined at /etc/kubernetes/manifests
cd /etc/kubernetes/manifests/
sudo mkdir ../manifests-backup
sudo mv ./* ../manifests-backup
# check it is empty
ls -la
# check etcd, apiserver, etc. is stopped
sudo crictl ps
# restore snapshot (we need to move the existing dirty etcd dir first, otherwise the tool will complain)
# need to match flags values from /etc/kubernetes/manifests/etcd.yaml, otherwise it will use the default which might not match the in-use values
sudo mv /var/lib/etcd /var/lib/etcd-dirty
sudo etcdctl snapshot restore /tmp/etcd-backup.db \
  --data-dir /var/lib/etcd \
  --name control-1 \
  --initial-cluster control-1=https://192.168.64.7:2380 \
  --initial-advertise-peer-urls https://192.168.64.7:2380 \
  --initial-cluster-token etcd-cluster
# check it actually wrote to the fs
sudo ls -la /var/lib/etcd
# now we can restart the k8s core services
sudo mv ../manifests-backup/* .
# check etcd is running now
sudo crictl ps
# check cluster state restored (dummy deploy and pod gone now)
kubectl get deploy
kubectl get pods
```
