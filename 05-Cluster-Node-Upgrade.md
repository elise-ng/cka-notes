# Cluster Node Upgrade

- Cluster can only be upgrade from one to another minor version, skipping minor version not supported (new!)
- Upgrade one by one: Kubeadm, control pane, then worker nodes

Always refer to: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

## Update kubeadm
`/etc/apt/sources.list.d/kubernetes.list`
```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.XX/deb/ /
```

```sh
# After updating apt source
sudo apt update
sudo apt-cache madison kubeadm
# We should see the newer version of kubeadm here
sudo apt-mark unhold kubeadm
sudo apt update
sudo apt install kubeadm='1.XX.Y-Z'
sudo apt-mark hold kubeadm
# Check version updated
kubeadm version
```

## Update control plane
```
# After kubeadm update
# Verify upgrade plan
sudo kubeadm upgrade plan
# Apply
sudo kubeadm upgrade apply v1.XX.Y
# Then, update CNI plugin if required (e.g. Calico in our case)
```

If there are more than one control plane nodes, upgrade rest:

```
# on the other control plane machines
# after updating apt source & kubeadm
sudo kubeadm upgrade node
```

## Update kubelet and kubectl on control plane node

```sh
# drain control plane node, including daemon sets (ignore the default filter)
kubectl drain CONTROL_NODE_NAME --ignore-daemonsets
# update kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt update
sudo apt install kubelet='1.XX.Y-Z' kubectl='1.XX.Y-Z'
sudo apt-mark hold kubelet kubectl
# restart kueblet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
# uncordon control plane node
kubectl uncordon CONTROL_NODE_NAME
# check status
kubectl get nodes
```

## Update worker node

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/

```sh
# On worker
# After updating apt source and kubeadm
# Update kubelet config
sudo kubeadm upgrade node
# Drain target node
kubectl drain WORKER_NODE_NAME --ignore-daemonsets
# update kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt update
sudo apt install kubelet='1.XX.Y-Z' kubectl='1.XX.Y-Z'
sudo apt-mark hold kubelet kubectl
# restart kueblet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
# uncordon worker node
kubectl uncordon WORKER_NODE_NAME
# check status
kubectl get nodes
```
