# Observability in Kubernetes

## Table of contents

1. [Readiness Probe](#1-readiness-probe)
   1. [HTTP Test](#1a-http-test)
   2. [TCP Test](#1b-tcp-test)
   3. [Exec command](#1c-exec-command)

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

## 1. Readiness Probe:

We can configure Readiness probes to ensure application inside a container is actually Ready to serve traffic.

There are different types of Readiness Probes.

- HTTP Test
- TCP Test
- Exec command

### 1a. HTTP Test:

When the container is created, Kubernetes does not immediately set the READY condition to True. Instead, it performs the
readiness test to ensure it's ready.

If we know that, application takes 3 seconds to warm up, we can add ```initialDelaySeconds``` to the readinessProbe. We can
also set the frequency of the test by setting ```periodSeconds``` field of readinessProbe. By default, Kubernetes does three
attempts for application to be ready. We can also set the threshold count for failed retries by setting ```failureThreshold```.

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
        initialDelaySeconds: 3
        periodSeconds: 5
        failureThreshold: 5
```

### 1b. TCP Test:

For TCP Test, Kubernetes ensures that the container is listening on specified port and then udpates the READY condition to
```True``` for service to route traffic.

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
        tcpSocket:
          port: 8080
```

### 1c. Exec command:

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
        exec:
          command:
            - cat
            - /api/is_ready.txt
```

## 2. Liveness Probe:

A Liveness probe can be configured within a container to periodically check whether the application is Live and healthy.
This is required for container to serve customer traffic if and only if application is healthy. This address the scenario
of container in READY state/condition even though the application is in hung state.

If the Liveness probe fails, the container is destroyed by Kubernetes and a new one is created.

The liveness probe configuration is exactly same as readinessProbe. Only change is the field name. Refer to below example.

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
      livenessProbe:
        httpGet:
          port: 8080
          path: /api/isready
        initialDelaySeconds: 3
        periodSeconds: 5
        failureThreshold: 5
```