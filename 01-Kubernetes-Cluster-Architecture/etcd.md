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