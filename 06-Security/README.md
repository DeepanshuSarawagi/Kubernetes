# Security concepts in Kubernetes

## Table of contents:

1. [Security Primitives](#1-security-primitives)
2. [Authentication-Basics](#2-authentication)

## 1. Security primitives:

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
- Communication between different application pods can be restricted using network policies.

## 2. Authentication:

We can authenticate the users to access kube-apiserver by updating the service or yaml file. Following flag has to be added
in the service/yaml file. ```--basic-auth-file=user-details.csv```.

To authenticate using basic credentials we can pass the credentials in curl command like this.

```shell
curl -k -v https://master-node-ip:6443/api/v1/pods -u "user1:xxxxxxxx"
```

We can also use the static token file instead of password file. Specify the file in kube-apiserver.service/yaml file like this
```--token-auth-file=user-token-file.csv```

To use token in curl command:

```shell
curl --request GET -sL \
     --url 'https://master-node-ip:6443/api/v1/pods'\
     --header "Authorization: Bearer xwgbrgbflsvbufhkbvusfbrgwrbgrwg"'
```