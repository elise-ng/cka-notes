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

