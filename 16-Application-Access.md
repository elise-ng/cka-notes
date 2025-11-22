# Application Access

## Networking Overview
- Between Containers: inter process communication
- Between Pods: network plugin e.g. calico
- Between Pods and Services: service resource
- Between External Users and Services: services + (ingress OR gateway api)
- External Network <-> Nodes <-> Cluster Netowrk (by api server) <-> Service <-> Pod Network (by network plugin)
- Service: NodePort or ClusterIP
- NodePort:
  - External User -> Load Balancer (outside k8s) -> NodePort on the machines -> Service
  - Works for any protocol
- Ingress:
  - HTTP / HTTPS only
  - Supports hostnames
  - External User -> Ingress (reverse proxy) -> Service

## Ingress
- Old solution for managing incoming traffic
- To be replaced by Gateway API soon
- Still in exam scope for now

## Network Plugins
- Implement network traffic between pods
- Provided by the k8s ecosystem, not built in
- e.g. Calico
- Add resources using CRD: `kubectl get crd`
- e.g. `kubectl get ippools` for calico

## Services
- Provide access to the pods
- If multiple pods used as endpoint, provides load balancing
- Types:
  - ClusterIP: service exposed internally, default value
  - NodePort: service exposed at each node's IP address
  - LoadBalancer: cloud provider offers a load balancer that route traffic to either NodePort or ClusterIP based services
  - ExternalName: service mapped to a DNS CNAME record
- `kubectl expose` to expose application via Pod / ReplicaSet / Deployment
- or `kubectl create service`

### Demo: Creating Services

```sh
kubectl create deploy webshop --image=nginx --replicas=3
kubectl get pods -- selector app-webshop -o wide
# 3 pods available
# expose port 80
kubectl expose deploy webshop --type=NodePort --port=80
kubectl describe svc webshop
# check NodePort assigned
```

