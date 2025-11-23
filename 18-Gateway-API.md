# Gateway API

- Replacement of Ingress
- Add more advanced features:
  - traffic management
  - more options in API resrouces
- Do not run Gateway API on a node that the already running an ingress controller
- Gateway API Controllers provided by k8s ecosystem (mirror to Ingress Controller), e.g. Nginx Gateway Fabric

## API Resources
- GatewayClass: represents Gateway Controller
  - `sepc.controllerName` to connect to a specific GatewayController
- Gateway: instance of traffic handling infrastructure
  - Multiple Gateway can connect to one Gateway Controller
  - At least one Gateway is required
  - `gatewayClassName` to connect to the GatewayController
  - `listeners` to specify which protocols should be serviced
- HTTPRoute: defines how traffic is routed to services
  - `spec.hostnames` identifies incoming requests
  - `parentRefs` connects HTTPRotue to a Gateway
  - `backendRefs` connects HTTPRoute to a Service

User -> (DNS) -> Gateway API Controller -> API Controller Pod -> Gateway Controller -> (gatewayClassName) -> Gateway -> (parentRefs) -> HTTPRoute -> (backendRefs) -> Service -> Pod

## Demo: Install Nginx Gateway Fabric

Ref: [](https://docs.nginx.com/nginx-gateway-fabric/install/helm/#deploy-nginx-gateway-fabric)

```sh
# Remove existing ingress controller
helm delete ingress-nginx -n ingress-nginx
# Install CRD
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v2.2.1" | kubectl apply -f -
# Install Gateway Controller
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway --set nginx.service.type=NodePort
kubectl wait --timeout=5m -n nginx-gateway deployment/ngf-nginx-gateway-fabric --for=condition=Available
# Check it is up and running
kubectl get pods,svc -n nginx-gateway
kubectl get gc
# Check Assigned NodePort
kubectl -n nginx-gateway describe svc ngf-nginx-gateway-fabric
# Set NodePort for http / https
kubectl -n nginx-gateway edit nginxproxy ngf-proxy-config
# Set specs.kubernetes.service.nodePorts, see below
# Create demo app
kubectl create deploy myapp --image=nginx --replicas=3
kubectl create service clusterip myapp --tcp=80:80
# Create gateway and httproute
kubectl apply -f gateway.yaml
kubectl apply -f httproute.yaml
# test, assuming /etc/hosts is edited to add the dns record
curl http://myapp.com:30080
```

`nginxproxy/ngf-proxy-config`
```yaml
apiVersion: gateway.nginx.org/v1alpha2
kind: NginxProxy
metadata:
  name: ngf-proxy-config
  namespace: nginx-gateway
spec:
  kubernetes:
    service:
      type: NodePort
      nodePorts:
        - port: 30080
          listenerPort: 80
        - port: 30443
          listenerPort: 443
```

`gateway.yaml`
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
  - name: https
    port: 443
    protocol: HTTPS
```

`httproute.yaml`
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: myapp-route
spec:
  parentRefs:
  - name: gateway
  hostnames:
  - "myapp.com"
  rules:
  - backendRefs:
    - name: myapp
      port: 80
```
