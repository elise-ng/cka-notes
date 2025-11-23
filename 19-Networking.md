# Networking

## Container Network Interface (CNI)

- Inferface for networking when starting kubelet on a worker node
- CNI ensures different network plugins from k8s ecosystem is interchangable
- `/etc/cni/net.d` for configs (some plugins may have extra configs or CRD)
- `kube-controller-manager` may define Pod CIDR with `--cluster-cidr` param which will be respected by plugins (e.g. Calico IPPool)

## Service Auto Registration
- coredns running at `10.96.0.10` for internal DNS
- Service resource will register with kubedns
- Pods are auto configed to use kubedns as DNS resolver, so they can resolve service by name
- For same namespace, use short hostname
- For another namespace, use `servicename.namespace.svc.clustername`
- `clustername` defined at coredns configmap, default is `cluster.local`
  - `kubectl get cm -n kube-system coredns -o yaml`

## NetworkPolicies
Ref: [](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- No restrictions to tarffic in k8s by default
- Use NetworkPolicies to secure traffic (some network plugins does not support this e.g. weave)
- If there is a NetworkPolicy, it is defauly deny & allow per rule
- Otherwise if there is no NetworkPolicy, all traffic is allowed
- direction: `ingress`, `egress`
- identifier: `podSelector`, `namespaceSelector`, `ipBlock`
- NetworkPolicies do not conflict, they are additive
- Attached to namespace, `podSelector: {}` for applying to all pods in namespace, or use a more specific pod selector

Example:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978

```


### Between Pods
- Use `podSelector.matchLabels`

### Between Namespaces
- NetworkPolicy is namespaced. If no pod selector are specified, it will apply to all pods in the namespace.
- For allow traffic from any namespace, `ingress.from.namespaceSelector: {}` (empty selector)
- Use immutable label `kubernetes.io/metadata.name` for specifying namespace by name
- Be careful if the selectors are in same statment or not:
### allow from namespace label alice AND pod label client
```yaml
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
      podSelector:
        matchLabels:
          role: client
```
### allow from _any pod_ in namespace label alice OR pods with label client _in local namespace_
```
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - podSelector:
        matchLabels:
          role: client
```
### allow from namespace label alice OR *global* pod with label client
```
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - namespaceSelector: {}
      podSelector:
        matchLabels:
          role: client
```
