# Cluster Maintenance in Kubernetes

## 1. OS upgrades:

Whenever a node goes offline due to issues or patches, kubernetes controller manager waits for 5 mins for pods to come online.
This timeout is called as ```pod-eviction-timeout```. When the node comes online, it comes up blank without any pods created
(provided no replica sets). To overcome this scenario where a node goes offline, we can manually drain the node. Draining
the node will ensure to place the pods in other healthy nodes. Thus, the node which requires any maintenance will be cordoned
or to say, no pods will be scheduled on the cordoned nodes. 

Once the maintenance activity is complete, we need to uncordon the node so that it is ready to have pods scheduled on it.

Commands

: ```kubectl drain node01 --ignore-daemonsets # To evict all the pods from this node```

: ```kubectl cordon node01  # To mark the node unschedulable```

: ```kubectl uncordon node01  # To mark the node schedulable```

Node draining will fail if there is any pod created on node which is not part of replica sets. We see below errors while trying
to drain the node.

```text
controlplane ~ ➜  kubectl drain node01 --ignore-daemonsets
node/node01 already cordoned
error: unable to drain node "node01" due to error:cannot delete Pods declare no controller (use --force to override): default/hr-app, continuing command...
There are pending nodes to be drained:
 node01
cannot delete Pods declare no controller (use --force to override): default/hr-app
```

This is because, when we try to drain the node, the pod which is created without any replicaset would be evicted and lost
forever. 

However, if we still want to drain that node, we can use the ```--force``` flag in the command.

```shell

controlplane ~ $kubectl drain node01 --ignore-daemonsets --force
node/node01 already cordoned
Warning: deleting Pods that declare no controller: default/hr-app; ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-6qvqr, kube-system/kube-proxy-k4wgp
evicting pod default/hr-app
pod/hr-app evicted
node/node01 drained

```
