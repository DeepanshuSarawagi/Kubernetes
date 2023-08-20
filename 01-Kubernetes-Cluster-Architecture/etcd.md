# Brief overview of ETCD in Kubernetes

- ETCD is a distributed, reliable key-value store that is simple, secure and fast.
- ETCD service by default runs on port 2379.
- It comes with a default client that ETCD control client. It is a cmd line client for etd.
  - ./etcdctl is the cmd line utility.
  - ```shell
    ./etcdctl set key1 value1  # Sets key value pair
    ./etcdctl get key1
    # ./etcdctl is supported to work with API version 2 and 3.
    # By default it is set to work with API version 2.
    export ETCDCTL_API=3  # To set etcdctl to work with API version 3.
    ./etcdctl put key1 value1  # Sets key value par in ETCD database if API version is set to 3
    ```
    
- etcd stores information regarding cluster such as,
  - Nodes
  - Pods
  - Configs
  - Secrets
  - Accounts
  - Roles
  - Role Bindings

- Whenever we run kubectl get command, the information we get is from etcd server.
- advertise client url is the address etcd service listens on.

## Topology:

### Stacked Topology:

Etcd runs on Kube master nodes where other core components run.

- Easier to setup
- Easier to manage
- Fewer servers
- Risk during failures

### External ETCD Topology:

ETCD runs on a dedicated servers separated from kube master nodes.

- High availability
- More servers
- Harder to setup

## ETCD in HA:

How does ETCD ensure data on all the nodes are consistent? We can write to any instance and read the data from any instance.

ETCD follows the leader/followers strategy. Out of all the nodes, one node will be elected as leader and rest will be followers.
READ operation can be done from any of the nodes, however, WRITE operation happens only on leader node. Leader node ensures
to process the WRITE operation and updates other nodes with the copy of the data.

If the WRITES come through any of the follower nodes, it forwards the operation to leader node internally. A WRITE can be
considered complete only when it receives consent from the other nodes in the cluster.