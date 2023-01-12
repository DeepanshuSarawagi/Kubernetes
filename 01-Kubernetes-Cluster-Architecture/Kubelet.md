#Brief overview of Kubelet

- Kubelet in the worker node registers the node with Kubernetes cluster.
- When it receives an instruction to load a container/pod on a node, it requests the container runtime engine to pull necessary
  images to spin up a pod/container.
- Kubelet then continues to monitor the state of the POD and reports it to the kube api server.
- kubeadm doesn't deploy kubelet, it needs to be downloaded and installed separately.