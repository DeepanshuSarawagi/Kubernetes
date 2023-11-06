# This is a brief document to understand about concept of Pods in Kubernetes

- Kubernetes does not deploy containers directly on the worker nodes.
- The Kubernetes are encapsulated into Kubernetes object known as Pods.
- A Pod is a single instance of an application
- A Pod is the smallest object you can create in Kubernetes.
- Pods usually have one-to-one relation with your containers running your application.
- You do not add additional containers to your existing pod to scale application.
- PODs usually have one-to-one relationship with container which runs the application.

Basic Kubectl commands to list and deploy pods:

How to spin-up a pod with nginx container
.\kubectl.exe run nginx --image=nginx

How to describe or view information about pods
.\kubectl.exe describe pods nginx

How to check on which node Pod is running and its internal IP
.\kubectl.exe get pods -o wide

How to check which api version and kind pods use
.\kubectl explain pods

Imperative command to create pods with labels
.\kubectl run nginx --image nginx:alpine --labels="app=webserver,tier=frontend"

Imperative command to create a pod and expose it as a clusterIp service
.\kubectl run nginx --image=nginx:alpine --port=80 --labels="tier=frontend" --expose=true

Note:

: Remember, you CANNOT edit specifications of an existing POD other than the below.

    spec.containers[*].image

    spec.initContainers[*].image

    spec.activeDeadlineSeconds

    spec.tolerations 