# Networking in Kubernetes

## Table of contents:

1. [Networking Basics](#1-networking-basics)
   1. [Switch](#1a-switch)
   2. [Router](#1b-router)
   3. [Gateway](#1c-gateway)
2. [DNS](#2-dns)
3. [Network Namespaces](#3-network-namespaces)
4. [Networking in Docker](#4-networking-in-docker)
5. [Container Network Interface](#5-container-network-interface)
6. [POD Networking](#6-pod-networking)
7. [IPAM](#7-ip-address-management)
8. [Service Networking](#8-service-networking)
9. [DNS in Kubernetes](#9-dns-in-kubernetes)
10. [Ingress](#10-ingress)
    1. [IngressController](#10a-ingress-controller)
    2. [Ingress Resources](#10b-ingress-resources)


## 1. Networking Basics:

This section covers networking basics. 

### 1a. Switch:

Let's consider there are two systems, A and B. Both these systems are connected to
each other using switch. To connect them to a switch we need an interface on each host. A switch enables communication between
systems in the same network.

### 1b. Router:

A Router helps two networks _connect_ together. A router has many switches and has multiple IPs assigned to enable communication
between multiple networks. A router is just any other device on the network.

### 1c. Gateway:

For a system to identify the correct router to route to the desired system is where gateway comes in. We need to configure
a gateway to allow systems to route to the other network. However, creating routes for every network could become a tedious task.

This is where a Default Gateway comes in.

A default gateway helps you reach the system outside your network when you are not aware of the route.

By default, in linux, packets are not forwarded from one interface to another. To allow packet-forwarding, we need to set
below setting to true/1.

```shell
[deepanshusarawagi@deepanshu ~]$ sudo cat /proc/sys/net/ipv4/ip_forward
1
```

To make the above changes permanent, we need to make changes in the ```/etc/sysctl.conf``` file.

```shell
cat /etc/sysctl.conf
net.ipv4.ip_forward = 1
```


## 2. DNS:

We can perform name resolution by adding the hostname and IP address entry in the ```/etc/hosts``` file. This is a good
solution for a small network. However, it becomes tedious to manage when the network grows. Hence, came the concept of DNS
server. We can point our hosts to DNS server to perform any resolution or look up of hosts on other networks. To point a
host to remote DNS server, we just need to add the host entry in ```/etc/resolv.conf``` file. However, remember that the
```/etc/hosts``` file will always act as local DNS server. Name resolution will happen first here, if any host is not found,
it would then forward to remote DNS server.

However, the above order of resolution can be changed by editing the entry in the ```/etc/nsswitch.conf``` file.

Let's say if we don't have any host entry added in local hosts file or remote DNS, then we can have remote DNS server forward
the name resolution to global **nameserver at address 8.8.8.8**. We just need to add this nameserver entry in ```/etc/resolv.conf```
file of remote DNS server.

## 3. Network Namespaces:

Network Namespaces in Linux are used by containers like Docker to isolate networks. When we create a network namespace, the 
containers can only see the processes run by it in its namespaces. Although, the underlying host can see all the processes
irrespective of who creates it.

Below commands should be run to create network namespace:

```shell
$ip netns add red
$ip netns add blue

# To list namespaces
$ip netns

# To list interfaces on the host
$ip link

# To list interfaces within the namespace
$ip netns exec red ip link 


# To connect two network namespaces using virtual cable
$ip link add veth-red type veth peer name veth-blue

# To attach network interface to appropriate namespace
$ip link set veth-red netns red
$ip link set veth-blue netns blue

# To assign IP addresses to these network interfaces in namespaces
$ip -n red addr add 192.168.15.1 dev veth-red
$ip -b blue addr add 192.168.15.2 dev veth-blue

# To bring up the interface in respective namespace
$ip -n red link set veth-red up
$ip -n blue link set veth-blue up
```

To create multiple interfaces in multiple namespaces within a host, we would need a virtual network/switch. Refer below
to create a virtual switch.

```shell
$ip link add v-net-0 type bridge

# Bring up the above virtual network
$ip link set dev v-net-0 up  # Think of it as interface for host and switch for namespaces

# To create a network interface in namespace
$ip link add veth-red type veth peer name veth-red-br
$ip link add veth-blue type veth peer name veth-blue-br

# Attach the interface to namespaces
$ip link set veth-red netns red            # To attach one end of interface with ns
$ip link set veth-red-br master v-net-0    # To attach other end of interface with virtual network
$ip link set veth-blue netns blue          # To attach one end of interface with ns
$ip link set veth-blue-br master v-net-0   # To attach other end of interface with virtual network


# Assign IP addresses to the interfaces in namespaces
$ip -n red addr add 192.168.15.1 dev veth-red 
$ip -n blue addr add 192.168.15.2 dev veth-blue

# Bring up the interfaces
$ip -n red link set veth-red up
$ip -n blue link set veth-blue up

# Finally assign the IP address to the bridge interface
$ip addr add 192.168.15.5/24 dev v-net-0   # Now the Linux host can reach the network in namespace red or blue via this bridge
```

## 4. Networking in Docker:

When running a docker container with none network, it cannot communicate with both docker host or outside world.

```shell
$docker run nginx --network none
```

If we run a container with host network, docker containers will share the network with hosts. Let's say if a container runs
on port 80, then no other container or process can use the same port.

```shell
$docker run nginx --network host
```

The third networking option is bridge. An internal private network which the docker hosts and containers attach to.
Whenever a container is created, docker creates a network namespace for it and then attaches it to the bridge. To connect
the network namespace within container and the bridge network, docker creates two virtual network interfaces. It attaches
those interfaces, one end to the container and other end to the bridge network.

## 5. Container Network Interface:

CNI defines how a network plug-in should be developed and how container runtime should invoke it. 

For Container Runtimes CNI specifies that:

- It must create network namespace.
- Identify network the container should attach to.
- Container Runtime to invoke network plug-in when container is added.
- Container Runtime to invoke network plug-in when container is deleted.
- JSON format of the network configuration.

For Plugins CNI specifies that:

- It must support command line arguments ADD/CHECK/DEL.
- It must support parameters container id, network ns etc.
- It must manage IP address assignment to PODs.
- It must return results in specific format.

Note:

: Docker does not support CNI. It supports its own Container Network Model. However, Kubernetes takes care of assigning
IP address to docker containers.

```shell
# Useful network commands to inspect ports and process listening on them

$netstat -nplt | grep 3574 # To find port for this pid
$netstat -anp | grep etcd  # To find active listen connections for this process
$ip address show type bridge
```

## 6. Pod Networking:

Below is the networking model for PODs.

- Every POD should have an IP address assigned.
- Every POD should be able to communicate with every other POD in the same node.
- Every POD should be able to communicate with every other POD on the other nodes without NAT.

## 7. IP address management:

IPAM or Ip Address Management in Kubernetes is about how an IP address assigned to a network and to a Pod. CNI owns IPAM
solution in the Kubernetes.

## 8. Service Networking:

A kubernetes service when created is accessible from all pods across all the cluster nodes. A service is hosted across the
cluster.
For a brief recap on different kinds of Kubernetes service, refer this [document](../11-Services).

### How are the Kube services get the IP address?

Every time a new service is created on the cluster, kube-proxy gets into action. A Kubernetes service gets IP address 
assigned from a pre-defined range by kube-proxy. The kube-proxy gets that IP address and creates a forwarding rule on 
each node saying any traffic coming on the IP of this service should go to the IP of the POD. We can inspect this IP 
address range in the kube-apiserver yaml definition.

Kube proxy uses different ways of creating these forwarding rules for every service.

- userspace
- ipvs
- iptables (default)

The proxy mode option can be set while configuring the kube-proxy service.

```shell
kube-proxy --proxy-mode [userspace | ipvs | iptables]

```

If no mode is selected, then iptables proxy mode is selected by default.

Note:
: This IP address CIDR range for service defined in the kube-apiserver definition should not overlap with CIDR range defined
in CNI for PODs. Look for IP address range in the following property ```--service-cluster-ip-range```.

We can see the rules created by kube-proxy using iptables nat table output.

```shell
iptables -L -t nat | grep <kube service name>
```
We can see the logs of these rules created by kube-proxy at ```/var/log/kube-proxy.log```

## 9. DNS in Kubernetes:

For any service created within the namespace, this is the DNS format.

```http://<name-of-the-service>.<namespace-name>.<svc>```

All the services and PODs are grouped together in the root domain called as cluster.local.

```http://<name-of-the-service>.<namespace-name>.<svc>.cluster.local```

Record for PODs are not created by default. However, once enabled, PODs get the following name format within the cluster.
It is a combination of IP of PODs separated by dash. For example.,

```shell
http://10-244-2-5.apps.pod.cluster.local/

# Wherein, "apps" is the namespace, "pod" subdomain is for grouping all PODs together and cluster.local is the root domain.
```
Kubernetes has its own DNS server as POD in kube-system namespace. It is called as ```CoreDNS```. Prior to kube version 1.12, this
was referred to as ```kube-dns```. ```CoreDNS``` uses a file to maintain all the records at ```/etc/coredns/Corefile```.
Within this file, there are a number of plug-ins configured to handle errors. 

Any POD which is not able to resolve the name such as www.google.com is forwarded to the nameserver configured in the CoreDNS
pod. When a CoreDNS component is deployed, a service is also created to make it available for other components in the kube
cluster. This service is known as ```kube-dns```.

Here is the sample Corefile.

```textmate
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}
```

## 10. Ingress:

Ingress resource helps you manage routing configuration, making all the application services accessible using a single externally 
accessible URL. Ingress is a _**layer 7 application lb**_ built within Kubernetes cluster that can be configured using native kubernetes
primitives just like any other Kube object. However, creating a Kube Ingress object alone isn't enough. We need to have a relevant IngressController
as well since Kubernetes Ingress object alone cannot handle SSL termination, load balancing requests et cetera. Here is 
where IngressController comes into the picture.

### 10a. Ingress Controller:

There are several solutions available which is supported by Kubernetes such as 

- Nginx (Supported by Kubernetes Project)
- HaProxy
- AWS LB controller
- Istio

We then configure set of routing rules which is the actual Ingress resource. Hence, URL routing and load balancing in Kubernetes
works in combination of IngressController solution with Ingress resource. Typically, Nginx IngressController with Kubernetes
Ingress object. Ingress Controller have additional intelligence built into them to monitor Kubernetes cluster for any
additional Kubernetes Ingress resources and configure the IngressController accordingly.

Refer example [Nginx Ingress definition](IngressController.yaml) here.

### 10b. Ingress Resources:

An Ingress Resource is set of rules and configurations applied on IngressController. Ingress Resources are created using 
definition file just like PODs, Deployments, Replica sets. Refer to the following [ingress.yaml](ingress.yaml)