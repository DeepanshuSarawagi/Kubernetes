#Brief overview of Kubernetes Scheduler

- Kube Scheduler is responsible for scheduling pods on nodes.
- It only decides which pods should go on which node which depends on pods resource requirements.
- Scheduler looks at each pod and tries to find the best node for it.
- Kubelet is the one which places pods on a node.