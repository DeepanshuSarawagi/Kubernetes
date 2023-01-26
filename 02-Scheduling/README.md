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