# Scheduling in Kubernetes

## 1. Manual scheduling
Every pod definition has a property called nodeName which isn't set by default. It gets added at kubernetes Live configuration
when pod is scheduled on node. The Kube-Scheduler goes through all the pods and checks for this property in the definition
file. If the property is not get, a scheduling algorithm is run which then identifies the best node to schedule the unscheduled
pod on it.

If there is no scheduler to monitor and schedule pods, we can then manually assign pods to nodes itself. We can achieve this
by setting the nodeName property in the pod-definition YAML file. This can only be set during the creation time. Kubernetes won't
allow us to override this property if the pod is already created. However, there is a way to achieve this as well.

We can create a Binding object and send a POST request to binding API.

Refer to [nginx.yaml](../Pods/nginx.yaml) for manually assigning the node.

## 2. Labels and Selectors

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
