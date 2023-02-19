# Cluster Maintenance in Kubernetes

## 1. OS upgrades:

Whenever a node goes offline due to issues or patches, kubernetes controller manager waits for 5 mins for pods to come online.
This timeout is called as ```pod-eviction-timeout```. When the node comes online, it comes up blank without any pods created
(provided no replica sets). To overcome this scenario where a node goes offline, we can manually drain the node. Draining
the node will ensure to place the pods in other healthy nodes. Thus, the node which requires any maintenance will be cordoned
or to say, no pods will be scheduled on the cordoned nodes. 

Once the maintenance activity is complete, we need to uncordon the node so that it is ready to have pods scheduled on it. 