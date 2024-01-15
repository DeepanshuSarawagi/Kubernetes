# Observability in Kubernetes

## Table of contents

There are different states of PODs during its lifecycle.

- PODScheduled
- Initialized
- ContainersReady
- Ready

The Kubernetes service starts routing traffic to a POD as soon as it is in a ```Ready``` state. 

However, think of a scenario
where an application takes time to come up due to some dependencies. Kube service routing to a Ready state POD even though
the application is not up would lead to poor customer experience.

To overcome above issue, comes the concept of Readiness Probe.

## 1. Readiness Probe

We can configure Readiness probes to ensure application inside a container is actually Ready to serve traffic.

There are different types of Readiness Probes.

- HTTP Test
- TCP Test
- Exec command

### 1a. HTTP Test:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readinesspod
  namespace: default
spec:
  containers:
    - name: readinesspod
      image: nginx
      imagePullPolicy: Always
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          port: 8080
          path: /api/isready
```