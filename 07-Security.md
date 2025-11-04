# Security

## PKI

- k8s authenticates using PKI
- CA for the cluster for signing certs
- Cert (public key) and Key (private key) for both server and clients
- kubectl commands is signed
- alternative: curl -> (plaintext) -> kube-proxy -> (signed) -> kube api
- enables user auth & RBAC

## SecurityContext

- defines privilege and access control for Pods / Containers
- Options:
  - UID and GID
  - SELinux labels
  - Linux Capabilities, AppArmor, Seccomp
  - AllowPrivilegeEscalation setting
  - runAsNonRoot setting
- `kubectl explain pod.spec.securityContext`, `kubectl explain pod.spec.containers.securityContext`
- Refer to docs for example during exam

## Users, Services Accounts
- k8s api does not natively define users to auth and authz
- users: defined by external identity provider
  - X.509
  - OpenID (e.g. Google, AD)
- service account: used by pods to access kube api
  - all pods have default service account for minimal access
  - we can create specific service account for extra access

## RBAC
- Role:
  - Verb: Create, List, etc.
  - Resource: Pods, Deployments, etc.
- RoleBinding:
  - Binds users / service accounts to roles
- `kubectl create role`, `kubectl create rolebinding`, `kubectl create serviceaccount`

### Demo
- Default service account token within pod: `/run/secrets/kubernetes.io/serviceaccount/token`
- Use the token in `Authorization: Bearer <token>` header

```sh
kubectl run mypod --image=alpine -- sleep 3600
# now `kubectl get pod mypod -o yaml` will show the pod is using `serviceAccount: default`
kubectl exec -it mypod -- sh
# in pod:
apk add --update curl
# try access api without token
curl https://kubernetes/api/v1 --insecure
# this will give 403 forbidden
# try access api with token
TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1 --insecure
# this will give list of apis available
# try api
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods --insecure
# this will give user system:serviceaccount:default:default cannot list resource

# now try create role and binding
# on operator machine:
kubectl create role list-pods --resource=pods --verb=list
kubectl create serviceaccount mysa
kubectl create rolebinding list-pods --role=list-pods --serviceaccount=default:mysa
kubectl apply -f mysapod.yaml
kubectl exec -it mysapod -- sh

# in new pod:
apk add --update curl
TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods --insecure
# this now works
```

`mysapod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysapod
spec:
  serviceAccountName: mysa
  containers:
  - name: alpine
    image: alpine:latest
    command:
    - "sleep"
    - "3600"
```

