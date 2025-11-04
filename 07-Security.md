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

### ClusterRoles, ClusterRoleBinding
- `Role` is scoped to namespace, `ClusterRole` applys to whole cluster
- use `ClusterRoleBinding` to provide access to users / service accounts

## Demo: RBAC for Pods
- Default service account token within pod: `/run/secrets/kubernetes.io/serviceaccount/token`
- Use the token in `Authorization: Bearer <token>` header
- There are default cluster roles: `kubectl get clusterroles` (e.g. `cluster-admin`)

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

## Demo: RBAC for Users
(Out of scope for CKA, in scope for CKS)
```sh
# create namespaces
kubectl create ns students
kubectl create ns staff
# create local linux users
sudo useradd -m -G sudo -s /bin/bash alice
sudo passwd alice
su - alice
# as alice
# create private key and cert signing request
openssl genrsa -out alice.key 2048
openssl req -new -key alice.key -out alice.csr -subj "/CN=alice/O=k8s"
# in real life csr is sent to admin for signing, using sudo here to simplify
sudo openssl x509 -req -in alice.csr -CA /etc/kuberbetes/pki/ca.crt
sudo openssl x509 -req -in alice.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out alice.crt -days 1800
# setup kubectl to use the user cert we created
# at /home/alice
mkdir .kube
sudo cp -i /etc/kubernetes/admin.conf .kube/config
sudo chown -R alice:alice .kube
kubectl config set-credentials alice --client-certificate=alice.crt --client-key=alice.key
kubectl config set-context alice-context --cluster=cka-cluster --namespace=staff --user=alice
kubectl config use-context alice-context
# try the user
kubectl get pods
# we will have forbidden error
# assign role now
exit
# as operator
kubectl create role staff -n staff --verb=get,list,watch,create,update,patch,delete --resource=deployments,replicasets,pods
kubectl create rolebinding -n staff staff-role-binding --user=alice --role=staff
# try the user again
su - slice
# as alice
kubectl get pods
kubectl create deploy nginx --image=nginx
# this should work now

# extra: view only role
kubectl get all -n default
# this will be forbidden currently
exit
# as operator
kubectl create role viewers -n default --verb=list,get,watch --resource=deployments,replicasets,pods
kubectl create rolebinding -n default viewers-role-binding --user=alice --role=viewers
# try the user again
su - slice
# as alice
kubectl get all -n default
# alice can read pods now as permitted, and forbidden from reading other resources
```
