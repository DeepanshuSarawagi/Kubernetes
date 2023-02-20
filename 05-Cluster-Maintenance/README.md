# Cluster Maintenance in Kubernetes

## Table of contents:

1. [OS Upgrades](#1-os-upgrades-)
2. [Cluster Upgrades](#2-cluster-upgrades-)
   1. [Upgrade process](#2a-upgrade-process-)
3. 

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

## 2. Cluster upgrades:

Basic info
: Every kubernetes components has it own release version. Since Kubernetes as a product has several components tied together, 
the main component of it is kube-api-server. Thus, none of the other component version should be higher than kube-api-server.

: For example, if kube-api-server is at v1.25, controller-manager and kube-scheduler can be at v1.24 (x-1) and kubelet and kube-proxy
can be at v1.23 (x-2). 

: However, in case of kubectl, it can be at v1.26 (x+1), v1.25 (x) or at 1.24 (x-1).

: Kubernetes supports upto version x-2 from its current stable release. Let's say, if kubernetes has released v1.26, it supports
components until v1.24.

If we have bootstrapped the cluster using kubeadm tool, following commands are useful for cluster upgrades.

```shell
$kubeadm upgrade plan

$kubeadm upgrade apply
```

### 2a. Upgrade process: 

UUpgrade process requires two major steps.

- First, upgrade the master/controlplane node.
- Second, upgrade the worker nodes.

For detailed steps on upgrading clusters using kubeadm, follow these [instructions](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/).

Note

: To reduce the time for upgrade using kubeadm, we can run the following command to pre-pull the images ```kubeadm config images pull```.

