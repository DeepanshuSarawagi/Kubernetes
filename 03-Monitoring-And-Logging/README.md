# Monitoring and Logging in Kubernetes

## Table of contents

1. [Metrics Server](#1-metrics-server-)
    1. [How does Metrics server work](#1a-how-does-metrics-server-work)
2. [Logging](#2-logging-)




## 1. Metrics Server:

Metrics server receives metrics from each of the kubernetes nodes or pods, aggregates them and stores them in-memory. This
is an in-memory monitoring solution. 

### 1a. How does metrics server work?

Kubelet is an agent which runs on every kube-node. It has a subcomponent called cAdvisor. cAdvisor is responsible for retrieving
performance metrics from pods and exposes them through kubelet api to metrics server. Once metrics are available, we can
run following command - ```kubectl top node``` to check the memory and CPU utilization of each node. 

We can also view performance metrics of pods using ```kubectl top pod``` command.

## 2. Logging:

Use below command to view logs of kubernetes pod.

```kubectl logs -f pod_name -c container_name```