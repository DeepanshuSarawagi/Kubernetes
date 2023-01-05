# Cluster Architecture in Kubernetes

- Worker nodes are responsible for hosting applications which runs Kubernetes pods.
- Master node can manage, plan, schedule and monitor worker nodes.
- ETCD is a database which stores information in key/value pair.
- Scheduler is responsible for identifying the right node to place a container based on container resource requirements
  and worker node capacity, any other policies such as taints or tolerations and node affinity rules.
- Kube Node Controllers takes care of nodes. Replication Controllers is responsible for replication of containers.
- Kube API server is the primary management component of Kubernetes cluster. It is responsible for orchestrating management
  operations. 
  - It exposes kubernetes api which allows users to perform management operations on the cluster.
- Container Runtime Engine is required to be installed on the Kubernetes cluster.
- A kubelet is an agent that runs on each node in a cluster. It listens for instructions from kube API server and deploys or destroys
  containers on the cluster as required.
- The Kube API server periodically fetches reports from the kubelet to monitor the status of nodes and containers on them.
- kube proxy enables the communication between worker nodes. Necessary rules will be in place to allow communication between
  different worker nodes.