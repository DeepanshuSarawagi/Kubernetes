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

### Leader election process:

ETCD implements distributed consensus using RAFT protocol. RAFT algorithm uses random timers for initiating requests. For
example, a random timer is kicked off on all the nodes. The first one to finish the timer sends out a request to the other
nodes requesting permission to be the leader. The other nodes respond the request with a vote and the requesting node assumes
the leader role.

Now, the newly elected leader sends notification to other master nodes at regular intervals that it is continuing to assume
the role of leader. In case, if other nodes do not receive the notification due to it leader node being down, the other master
nodes initiate the leader re-election process and a new leader is elected.

### Quorum:

Quorum is the minimum number of nodes that must be available. In a cluster network of three nodes, the quorum value is N/2 + 1.

| Instances | Quorum | Fault Tolerance |
|:----------|:-------|:----------------|
| 1         | 1      | 0               |
| 2         | 2      | 0               |
| 3         | 2      | 1               |
| 4         | 3      | 1               |
| 5         | 3      | 2               |
| 6         | 4      | 2               |
| 7         | 4      | 3               |

It is always recommended to consider odd number of clusters since fault tolerance is high when compared to even number of
nodes.