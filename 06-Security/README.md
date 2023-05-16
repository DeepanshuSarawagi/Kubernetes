# Security concepts in Kubernetes

## Security primitives:

- kube-apiserver is at the center of all operations within kubernetes.
- Who can access the kube-apiserver is defined by the Authentication mechanism.
- We can have following types of authentication for accessing kube-apiserver.
  - Username and password on files
  - Username and tokens
  - Certificates
  - External Authentication Providers - LDAP
  - Service accounts
- Once the access is given, what can they do is defined by the Authorization mechanism.
- We can have following types of authorization for accessing kube-apiserver.
  - RBAC(Role Based Access Control) Authorization
  - ABAC(Attribute Based Access Control) Authorization
  - Node Authorization
  - Webhook mode
- All communication between kube-apiserver and other components such as etcd server, kube-proxy, kube-controller manager,
  kube-scheduler are encrypted using TLS certificates.