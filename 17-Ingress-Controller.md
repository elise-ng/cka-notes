# Ingress Controller

- Ingress is an api objeect that manages external access to services
- Works with external DNS to provide URL-based access
- Two Parts: external load balancer + API resource for looking up backend pods
- Ingress load balancers provided by k8s ecosystem, e.g. nginx
- Uses `selectorlabel` in services to connect to the pod endpoints
- Traffic routing controlled by rules set in Ingress resource
- Common features provided:
  - Provides externally reachable URLs
  - load balance traffic
  - terminates SSL/TLS
  - name based virtual hosting

## Demo: Install nginx ingress controller

Ref: [](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)

```sh
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
# check pod created
kubectl get pods -n ingress-nginx
# create dummy service
kubectl create deploy nginxsvc --image=nginx --port=80
kubectl expose deploy nginxsvc --port=80
kubectl describe svc nginxsvc
# create ingress
kubectl create ing nginxsvc --class=nginx --rule=nginxsvc.info/*=nginxsvc:80
# test by accessing via localhost:8080 -> ingress controller
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
# ctrl-z bg
# add nginxsvc.info in /etc/hosts
curl nginxsvc.info:8080
# ingress will serve the service correctly
```

User -> (DNS) -> ingress service -> ingress pod -> (ingress rule) -> application service -> application pods

## Configurations
- Ingress rules match incoming traffic and connects it to a service and port
- we can route paths to different servies, or virtual hosts to different services
  - e.g. `--rule="/=webshop:80" --rule="/hello=newdep:8080"` routes according to path
- different ingress controllers can be hosted, specify with `IngressClass`
- `ingressclass.kubernetes.io/is-default-class: true` annotation on the IngressClass crd to make it default

## Extra: Port Forwarding
- useful for troubleshooting, connects to pod's port directly
- `kubectl port-forward mypod 1234:80`: forwards local port 1234 to pod port 80
- (ctrl-z + bg) OR run command with `&` at the end to run in background
