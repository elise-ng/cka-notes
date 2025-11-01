# Initialize nodes

## Background
3 nodes: 1 control plane, 2 workers
On VM: 2 CPU 4G RAM, 64G SSD

## Steps
0. Create VM with Ubuntu 24.04.3
1. Setup containerd:
Ref: [k8s docs - install kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Ref: [containerd docs - install](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)

```sh
# enable ip forwarding
# in /etc/sysctl.conf,
# uncomment net.ipv4.ip_forward=1
# uncomment net.ipv6.conf.all.forwarding=1
sudo sysctl -p /etc/sysctl.conf
# disable swap
sudo swapoff -a
# then, in /etc/fstab, remove swap mounting on boot
sudo rm /swap.img
# install containerd
wget https://github.com/containerd/containerd/releases/download/v2.1.4/containerd-2.1.4-linux-arm64.tar.gz
sudo tar Cxzvf /usr/local ./containerd-2.1.4-linux-arm64.tar.gz
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
# install runc
wget https://github.com/opencontainers/runc/releases/download/v1.3.2/runc.arm64
sudo install -m 755 runc.arm64 /usr/local/sbin/runc
# install cni plugin
wget https://github.com/containernetworking/plugins/releases/download/v1.8.0/cni-plugins-linux-arm-v1.8.0.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-arm-v1.8.0.tgz
```
2. Setup kubeadm:
```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```
3. Setup control plane
To get default config file template, `kubeadm config print init-defaults > kubeadm-init-config.yaml`
### kubeadm-init-config.yaml
```yaml
apiVersion: kubeadm.k8s.io/v1beta4
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.64.7
nodeRegistration:
  name: control-1
---
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
clusterName: cka-cluster
```
Init:
```sh
sudo kubeadm init --config ./kubeadm-init-config.yaml --dry-run
# if ok,
sudo kubeadm init --config ./kubeadm-init-config.yaml
```
