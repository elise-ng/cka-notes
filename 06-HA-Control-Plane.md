# High Availability Control Plane

Options:
- Stacked control plane nodes: less infra, etcd members and control plane nodes are co-located
- External etcd cluster: more infra, control plane nodes and etcd members separated


Requirements:
- Need a load balancer for clients to connect to
- (Setting up a LB is out of scope for CKA exam)

<img width="500" src="https://github.com/user-attachments/assets/f6b67e38-a30d-400c-8df8-b02c04c52115" />
<img width="500" src="https://github.com/user-attachments/assets/1b0c9516-2d7c-43ab-b838-4b37622fa8ff" />
