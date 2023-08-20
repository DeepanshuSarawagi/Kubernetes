# Brief overview of Kubernetes Scheduler

- Kube Scheduler is responsible for scheduling pods on nodes.
- It only decides which pods should go on which node which depends on pods resource requirements.
- Scheduler looks at each pod and tries to find the best node for it.
- Kubelet is the one which places pods on a node.

# Multi-Node architecture

Let's say that we have a multi-node master setup in our environment, that consists of multi instances of KubeScheduler/
Controller Manager/Kube API Server running. How does Kube Scheduler decide when to act on? They have to run on active/standby
mode.

So who decides which node should be active and which node should be standby? 

This is achieved through leader election process.

- When a Kube controller manager is created, we may specify a ```--leader-elect true```. 
- With this option when controller manager process starts, it tries to gain a lease and obtains a lock on kube object endpoint. 
- Whichever process first obtains this lock becomes the master node. 
- The duration of the leader elect lease operation is set to 15s; ```--leader-elect-lease-duration 15s```. 
- The active process who is the leader then renews the lease every 10s; ```--leader-elect-renew-deadline 10s```. 
- Kube controller manager process running on every node tries to become leader every 2s; ```leader-elect-retry-period 2s```.
- This way if one node fails, the process running on other node can acquire the lock and become the leader.
- The scheduler follows the similar approach and has similar cmd line options.