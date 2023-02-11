# Scheduling in Kubernetes

## Table of contents

1. [Manual Scheduling](#1-manual-scheduling-)
2. [Labels and Selectors](#2-labels-and-selectors-)
3. [Taints and Tolerations](#3-taints-and-tolerations-)
4. [Node Selectors](#4-node-selectors-)
5. [Node Affinity](#5-node-affinity-)
6. [Resource Requests and Limits](#6-resource-requirements-and-limits-)
7. [DaemonSets](#7-daemonsets-)
8. [Static Pods](#8-static-pods-)
 

## 1. Manual scheduling:
Every pod definition has a property called nodeName which isn't set by default. It gets added at kubernetes Live configuration
when pod is scheduled on node. The Kube-Scheduler goes through all the pods and checks for this property in the definition
file. If the property is not get, a scheduling algorithm is run which then identifies the best node to schedule the unscheduled
pod on it.

If there is no scheduler to monitor and schedule pods, we can then manually assign pods to nodes itself. We can achieve this
by setting the nodeName property in the pod-definition YAML file. This can only be set during the creation time. Kubernetes won't
allow us to override this property if the pod is already created. However, there is a way to achieve this as well.

We can create a Binding object and send a POST request to binding API.

Refer to [nginx.yaml](../Pods/nginx.yaml) for manually assigning the node.

## 2. Labels and Selectors:

Labels are properties attached to the kube objects. Selectors are used to filter the different kube objects.

Annotations are used to record other details for informatory purpose. 

```yaml
# Example of using labels in Kubernetes objects
apiVersion: v1
kind: Pod
metadata:
  name: webserver1
  labels:
    type: webserver
    tier: frontend
    app: ui
spec:
  containers:
    - name: webserver1
      image: nginx:alpine
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    type: webserver-rs
  annotations:
    buildVersion: "1.1"
spec:
  selector:
    matchLabels:
      tier: frontend-2
      app: login-page
  template:
    metadata:
      labels:
        tier: frontend-2
        app: login-page
    spec:
      containers:
        - name: nginx
          image: nginx
  replicas: 3
```
```shell
# Example of filtering objects using selector
$kubectl get pods --selector tier=frontend
```

## 3. Taints and Tolerations:

Taints and Tolerations are used to set restrictions on what pods can be scheduled on a node. <mark>Taints are set on nodes and tolerations
are set on pods.</mark>

```kubectl taint nodes node01 app=auth:NoSchedule|PreferNoSchedule|NoExecute```

This is the command to taint a node. As you can see from the above command, you can taint a node with three different taint
effects.

NoSchedule
: Any POD which do not tolerate this taint, will not be scheduled on this node.

PreferNoSchedule
: System/Kube Scheduler will try not to schedule a pod on this tainted node but it does not guarantee.

NoExecute
: No new pods would be scheduled on this tainted node. Any pods which have already been created on this node would be evicted 
  if they do not tolerate the taint. (PODs which were created before the node was tainted)


```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    tier: backend
spec:
  containers:
    - name: tomcat-instance1
      image: tomcat
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "auth"
      effect: "NoSchedule"
```
Once these tolerations are added, only pods which can tolerate the taints specified will be scheduled on matching tainted nodes.

## 4. Node Selectors:

We can set limitations on pod to only run on desired nodes. This can be achieved by using nodeSelector field in the spec section
of pod definition. The key-value pair under nodeSelector as seen in [node-selector-example.yaml](node-selector-example.yaml) 
is defined as labels for a node. The scheduler uses these labels to match and identify the nodes to place the pods on.

We can use below command to label a node.

```shell
$kubectl label nodes <node-name> <label-key>=<label-value>

# example

$kubectl label nodes node01 size=Large
```

## 5. Node Affinity:

Let's say if we have a complex requirement where we want to place pods on node of size medium or large, or lets say if do not
want to place pods on a node which is small, then node selector makes it difficult to achieve it. Hence, comes the concept of
node affinity.

With Node Affinity, we get the flexibility to place pods on a desired node even if the node requirement is complex.

### 5a. Node Affinity types:

requiredDuringSchedulingIgnoredDuringExecution:
: The scheduler will mandate that any new pods be scheduled on the given affinity rules. But for those pods which are already
  scheduled, this rule can be ignored, and will continue to run.

preferredDuringSchedulingIgnoredDuringExecution:
: The scheduler will not mandate that any new pods be scheduled on the given affinity rules.

requiredDuringSchedulingIgnoredDuringExecution:
: The scheduler will mandate that any new pods be scheduled on the given affinity rules. And for those pods which are already
  scheduled, this rule will be applied as well. Let's say that if a change is made to a node that do not meet affinity rules,
  then the scheduled pods will be evicted.

| Affinity Type | DuringScheduling | During Execution |
|:-------------:|:----------------:|:----------------:|
|    Type 1     |     Required     |     Ignored      |
|    Type 2     |    Preferred     |     Ignored      |
|    Type 3     |     Required     |     Required     |

Refer [node-affinity-example.yaml](node-affinity-example.yaml) to see how we can take advantage of affinity rules to schedule
pods on a desired node.

## 6. Resource Requirements and Limits:

Every pod requires a certain amount of CPU and memory. By default, every container in a pod requests at least 0.5 CPU and
256Mi of memory. We can specify this by creating the LimitRange object as defined in [limit-range](limit-range.yaml).

To explain CPU limits, 1 CPU count in pod-definition.yaml represents 1 AWS vCPU/1 GCP Core/1 Azure Core/1 Hyperthread.
And 1Mi - 1 Mebibytes is equivalent to 1,048,576 bytes.

**By default, Kubernetes sets a limit of 1CPU and 512Mi a container can utilize on a node. We can override this setting as 
defined in [pod-resource-example.yaml](pod-resource-example.yaml) by setting limits under resources section.**

## 7. DaemonSets:

Daemon Sets are like replica sets which runs multiple instances of pods but it runs one copy of pod in every node of a cluster.
Whenever a new node is added to a cluster, a replica of pod is added to that node. DaemonSet ensures, a copy of pod is always
available on every node of the cluster. 

Uses cases:

: Let's say if we want to deploy a monitoring agent on every node of a cluster then daemon set is a perfect fit for this
requirement.
Another use case is Kube-Proxy which is deployed as daemonset on every node in Kube cluster.
One more use is networking agent, Weave net networking solution requires an agent to be up and running on every node of
kube cluster. Hence, the weavenet networking agent is a perfect fit for this use case.

For example definition, refer [daemonset-example.yaml](daemonset-example.yaml).

```shell
# Command to view daemonsets in a cluster

$kubectl get daemonsets
$kubectl describe daemonsets
```

Daemonset uses **node affinity** to schedule pods on every node.

## 8. Static Pods:

In a normal scenario, where we have a Kubernetes master which hosts kubernetes components such as etcd-controller, kube-scheduler,
kube-api server et cetera. Every worker has a kubelet installed which waits for instructions from kube-api server to install
a pod. As we know that, kube-scheduler is responsible for scheduling the pods on right nodes. However, lets consider a scenario,
wherein kube-master is not available. So, how does kubelet install pods in such cases?

Kubelet, periodically checks for any pods-definition yaml files under this directory - ```/etc/kubernetes/manifests```.
Kubelet will try to create pods using any yaml file found in this directory. These pods are called as **static pods**. Any
changes made to these yaml files are picked by kubelet and then the respective change is applied to those corresponding
static pods.

**Note:**

- You can only create pods this way. You cannot create Deployments or Replica Sets or services using static pods concept.

- Kubelet works at pods level, which is why it is able to create static pods using this way.

- The directory which kubelet looks for is called as **--pod-manifest-path**. It can be configured to any location in kubelet.service.

- There is another way to configure --pod-manifest-path. You can create a config.yaml with kubelet properties and refer that
  yaml using --config parameter in kubelet.service. This config.yaml in turn can have manifest path configured using
  staticPodPath option. This approach is used by kubeadm to install kube clusters.

```shell
#kubelet.service
ExecStart=/usr/local/bin/kubelet \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --config=kubeConfig.yml \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2

```
```yaml
#kubeConfig.yaml
staticPodPath: /etc/kubernetes/manifests
```
Use Case:

: One use case we can think of is since kubelet is independent of master to create static pods, we can configure the control
plane components as static pods on every master node. This is how kubeadm configures kube components when it is used to bootstrap
a cluster.
That's why, when kubeadm is used, all the core kube components are installed as pods in kube-system namespace instead of 
installing them as a service.

## 9. Multiple Schedulers:

A kubernetes cluster can have multiple schedulers at a time. We can create deployments or pods instructing to use custom
scheduler configured by us to schedule pods. Default scheduler is named as ```default-scheduler```.

## 10. Scheduler Profiles:

The PODs which are waiting to be created first end up in scheduling queue. At this stage, PODs are sorted based on the priority
defined (Refer [high-priority](high-priority.yaml)). Once the priorityClass is created, you need to define it in 
pods definition file [pods-high-priority](pods-high-priority.yaml).


