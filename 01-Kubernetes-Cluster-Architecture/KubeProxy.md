# Brief overview of kube proxy

- A pod network is spanned across all the kubernetes nodes to establish communication between pods.
- Kube proxy is a process which runs on every node in kubernetes cluster.
- Its job is to look for new services and every time a new service is created, it creates appropriate rules on each node
  to forward traffic to services to route the traffic to backend pods.
- kube-proxy is deployed as daemonset on each node.