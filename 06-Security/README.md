# Security concepts in Kubernetes

## Table of contents:

1. [Security Primitives](#1-security-primitives)
2. [Authentication-Basics](#2-authentication)
3. [TLS Basics](#3-tls-introduction)
   1. [Server certificates in Kube](#3a-server-certificates-in-kube)
   2. [Client certificates in Kube](#3b-client-certificates-in-kube)
   3. [Generating certificates in Kube](#3c-generating-certificates-in-kubernetes)
   4. [Certificate API](#3d-kubernetes-certificate-api)
4. [Kubeconfig](#4-kubeconfig-)
5. [API Groups](#5-API-Groups)
6. [Authorization](#6-authorization)
   1. [RBAC](#6a-rbac)
7.

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

## 3. TLS Introduction:

There are two kinds of certificates:

- Server certificates.
- Client certificates.

Kubernetes cluster requires certificates to secure all the communication between its different components.

### 3a. Server certificates in Kube:

Kube-api server exposes HTTPS service that other kube components use to manage the cluster.

Another kube-component which uses server certificates is etcd-server. The etcd server stores information of the cluster, hence
it requires certificate key pair for itself.

Kubelet server also exposes HTTPS API endpoint on worker nodes that the kube-apiserver talks to. This also requires server
certificates.

### 3b. Client certificates in Kube:

The clients who access the kube-apiserver are the users through Kubectl Rest API.

Below are several clients within Kubernetes which requires client certificates:

- Kube's administrators to access kube-apiserver to control server through Kubectl Rest API.
- Kube-scheduler talks to kube-apiserver to schedule pods on right worker nodes.
- Kube-Control manager also accesses kube-apiserver hence it requires it's own client certs and key.
- Kube-proxy requires client certificate to authenticate with kube-apiserver.
- Kube-apiserver is the only component which talks to etcd-server. Hence, it requires it's own client certs and key for communication.

### 3c. Generating certificates in Kubernetes

#### CA Certificates:

We are going to create self-signed CA certs. For all certs, we will use this CA key-pair to sign them.

```shell
openssl genrsa -out ca.key 2048

openssl req -new -key ca.key -subj "\CN=Kube-CA" -out ca.csr

openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

#### Client certificates:

This certificate will be used by admin user to authenticate kube-control with.

Name must be prefixed for following components while creating client certificates to authenticate itself with kube-apiserver.

- Kube-scheduler is part of the system component hence its name must be prefixed with keyword system in the certificate.
- Kube-controller-manager is part of the system component hence its name must be prefixed with keyword system in the certificate.
- Kube-proxy is part of the system component hence its name must be prefixed with keyword system in the certificate.

```shell
openssl genrsa -out admin.key 2048

openssl req -new -key admin.key -subj "\CN=Kube-admin" -out admin.csr

openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt

# We have signed the admin certificate using CA key pair which makes it a valid certificate
```

#### Server certificates:

Name must be prefixed for following components while creating server certificates to authenticate itself with etcd-server.

- 

```shell
openssl genrsa -out etcd-server.key 2048

openssl req -new -key etcd-server.key -subj "\CN=etcd-server" -out etcd-server.csr

openssl x509 -req -in etcd-server.csr -CA ca.crt -CAkey ca.key -out etcd-server.crt

# We have signed the admin certificate using CA key pair which makes it a valid certificate
```

kube-apiserver certificate is the most important certificate in the kubernetes environment. Kube-apiserver component goes by
many component, hence, all the aliases (SANs) must be added in the certificate signing request using [apiserver.cnf](apiserver.cnf).

```shell
openssl genrsa -out apiserver.key 2048

openssl req -new -key apiserver.key -subj "\CN=kube-apiserver" -out kube-apiserver.csr --config apiserver.cnf

openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -out apiserver.crt
```

If something is wrong with kube-apiserver certificate or etcd server certificate, then kubectl commands may not work to troubleshoot.
In this case we need to inspect the logs of the container using docker commands.

### 3d. Kubernetes Certificate API:

Kubernetes has a builtin certificate API which can sign a certificate, issue the certificate and renew the certificate on our
behalf.

We can create a certificate signing request object. 

Steps to create CSR using API:

- Run openssl command to generate private key and CSR.
- Send the CSR to kubernetes API to get it signed using kube-apiserver-client.
  - Refer to [kubecsr.yaml](kubecsr.yaml) for a sample CSR object.
- Run below kubectl commands to get the certificate.

```shell
kubectl apply -f kubecsr.yaml

kubectl get csr # to view the csr
kubectl certificate approve <csr name>
```

Few things to note: 
: kube controller-manager has builtin certificate signer. In-order for us to use the kubectl to approve and sign the request,
we must ensure to pass on the following commands args to kube-apiserver configuration file.
cluster-signing-cert-file: /etc/kubernetes/pki/ca.crt
cluster-signing-key-file: /etc/kubernetes/pki/ca.key

Following [Kubernetes document](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user)
shows how to issue a certificate to a normal user.

## 4. Kubeconfig 

Every user using kubectl command to query any kube object needs to authenticate itself against kube-apiserver. And following
args needs to be specified in each API request which may get tedious for the user.

```shell
kubectl get pods --server my-server --client-key admin.key --client-certificate admin.crt --certificate-authority ca.crt
```

Hence, we can use a kubeconfig file to save all this information and pass it as a args reference like below.

```shell
kubectl get pods --kubeconfig config
```

By default, kubectl utility will look for a config file under ```~\.kube\config``` which allows us not to explicitly pass
it every time we run kubectl commands. This config file has information such as 

- clusters  # Different clusters created in an environment
- users     # Users with appropriate roles who has access to clusters
- contexts  # Which set of users has access to which cluster. Context naming convention is generally
user@cluster-name

You can specify multiple clusters and contexts in the same file. We can switch to different cluster by using below kubectl
commands.

```shell
kubectl config set-context --current <context-name>
```

To view current-context, run following command

```shell
kubectl config current-context
```

To view config file, we can run following command

```shell
kubectl config view
```

Sample [config](sampleConfig) file for reference.

You can also switch to different namespace by using following command.

```shell
kubectl config set-context --current --namespace <namespace>
```

## 5. API Groups:

We will look into API groups responsible for cluster functionality. There are two API gropups.

- Core group - /api
- Named group - /apis

We can list all the kubernetes APIs using following command:

```shell
kubectl api-resources -o wide

kubectl api-resources -o name
```

## 6. Authorization:

There are four kinds of authorization mechanism in Kubernetes.

- Node
- ABAC
- RBAC
- Webhook

Node:

: The kubelet accesses the kube-apiserver to read information about services, endpoints, nodes and pods. The Kubelet also
reports information to kube-apiserver such as nodes status, pods status, events. These requests are handled by a special authorizer
called Node Authorizer. Hence, kubelets should be part of system:nodes group and have name prefixed with system:node. Hence,
any user with this name and belonging to group is authorized by node authorizer.

ABAC:

: Attributes Based Access Control is where you associate a set of users or groups with set of permissions. Each user or group
has its own set of ABAC policies and kube-apiserver needs to be restarted for change in policy to take effect. Hence, it gets
difficult to manage. This is where RBAC comes in.

RBAC:

: Role Based Access Control is where you associate users/groups with roles. We create a role with set of permissions required
and associate subjets with it. Any change in role can be centrally managed and need not have to be done for every subject.

Webhooks:

: Webhook lets you manage the authorization externally and not through the built-in mechanisms. There are tools available for
kube-apiserver to decide which users are allowed to perform operations on kubernetes cluster.

### 6a. RBAC:

We can enable RBAC by creating a Role object. Once the desired [Roles](rbac.yaml) is ready, we can create the role using
kubectl command. Once the role is created, we need to map the user with this Role. This can be achieved by creating the
RoleBinding object. Once the desired [RoleBinding](role-binding.yaml) specification is ready, you can create it using kubectl
command.

Here are some kubectl commands to view roles and rolebindings.

```shell
kubectl get roles
kubectl get rolebindings

kubectl describe roles/developer
kubectl describe rolebindings/devuser-develop-role-binding
```

Once these objects are created, you can check relevant access without actually performing the action to test it.

```shell
$kubectl auth can-i create deployments 
no # returns no if you do not have access

$kubectl auth can-i create pods 
yes # returns yes if you have access

$kubectl auth can-i create pods --as dev-user
yes # returns yes if dev-user has access to create pods in default namespace

$kubectl auth can-i create pods --as dev-user --namespace test
no # returns no if dev-user does not have access to create pods in test namespace

```