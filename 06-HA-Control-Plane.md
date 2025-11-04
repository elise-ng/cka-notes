# High Availability Control Plane

Options:
- Stacked control plane nodes: less infra, etcd members and control plane nodes are co-located
- External etcd cluster: more infra, control plane nodes and etcd members separated

Requirements:
- Need a load balancer for clients to connect to (e.g. Keepalived+HAProxy)
- (Setting up a LB is out of scope for CKA exam)

<img width="500" src="https://github.com/user-attachments/assets/f6b67e38-a30d-400c-8df8-b02c04c52115" />
<img width="500" src="https://github.com/user-attachments/assets/1b0c9516-2d7c-43ab-b838-4b37622fa8ff" />

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

https://github.com/kubernetes/kubeadm/blob/main/docs/ha-considerations.md#options-for-software-load-balancing

Networking Setup: (Out-Of-Scope for CKA Exam)
- HAProxy as load balancer, distribute traffic to healthy api servers
- Keepalived provides floating IP that points to an active HAProxy node, failovers if one of the LB fails
- (on cloud env) Kube API endpoint points to the virtual ip of keepalived
- HAProxy+Keepalived runs on each of the control plane nodes (or 2+ dedicated machine)
- kubectl clients connect to VIP:6443
- verify connectivity with `nc <VIP> 6443`

Cluster Setup: (In-Scope for CKA Exam)
- Similar to basic setup, but we use the HA control plane endpoint

```sh
# init first control plane node
# always use dns for cloud env, can use virtual ip for demo env
sudo kubeadm init --control-plane-endpoint "<VIP>:6443" --upload-certs
# then, apply CNI plugin (e.g. calico)
# init second and third control plane node
# (cmd provided in output of first node init step)
sudo kubeadm join <...> --control-plane <...>
# check status
kubectl get nodes
# then, proceed to join worker nodes as normal
```
