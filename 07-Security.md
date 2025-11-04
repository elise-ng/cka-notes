# Security

## PKI

- k8s authenticates using PKI
- CA for the cluster for signing certs
- Cert (public key) and Key (private key) for both server and clients
- kubectl commands is signed
- alternative: curl -> (plaintext) -> kube-proxy -> (signed) -> kube api
- enables user auth & RBAC

##
