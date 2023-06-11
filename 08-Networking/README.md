# Networking in Kubernetes

## Table of contents:

1. [Networking Basics](#1-networking-basics)
   1. [Switch](#1a-switch)
   2. [Router](#1b-router)
   3. [Gateway](#1c-gateway)


## 1. Networking Basics:

This section covers networking basics. 

### 1a. Switch:

Let's consider there are two systems, A and B. Both these systems are connected to
each other using switch. To connect them to a switch we need an interface on each host. A switch enables communication between
systems in the same network.

### 1b. Router:

A Router helps two networks connect together. A router has many switches and has multiple IPs assigned to enable communication
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
## 2. DNS:

We can perform name resolution by adding the hostname and IP address entry in the ```/etc/hosts``` file. This is a good
solution for a small network. However, it becomes tedious to manage when the network grows. Hence, came the concept of DNS
server. We can point our hosts to DNS server to perform any resolution or look up of hosts on other networks.