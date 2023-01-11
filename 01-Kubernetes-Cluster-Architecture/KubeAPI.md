# Brief overview of Kube API server in Kubernetes

- When we run the kubectl command, kube control utility reaches the kube api server and authenticates the user.
- It then retrieves the data from etcd server and responds back the requested information.
- kube api server interacts directly with etcd datastore.