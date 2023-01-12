# Brief overview of Kube Controller

- Kube Controller manages various controller in Kubernetes such as 
  - Replication Controller
  - Scheduler Controller
  - Node Controller
  - Deployment Controller
  - Kube Controller
  - Namespace Controller
  - Endpoint Controller
  - PV-Binder-Controller
  - Service Account Controller
  - Job Controller
  - PV-Protection-Controller
  - Stateful Set
- A controller is a process that continuously monitors the state of a system in Kubernetes.
- For example, a Node Controller monitors the health status of nodes every 5 seconds (Node Monitor Period = 5s)
- If a node is unhealthy or if node controller stops receiving pings from a particular node, it makes it as unhealthy after
  40s (Node Monitor Grace Period = 40s)
- It then gives 5 mins for nodes to come up (POD Eviction timeout = 5m)
- If the node doesn't come up in the above specified interval, it removes the pods assigned to the node and assigns them to
  the healthy nodes if they are part of replica sets.
- Another controller is Replication Controller which ensure desired number of pods are available all the time.
- All these controller processes are packaged in a single process called Kubernetes Controller Manager
  - It can be installed using binaries. It is installed in kube-system namespace.
- /etc/kubernetes/manifests/kube-controller-manager.yaml is the manifest file.