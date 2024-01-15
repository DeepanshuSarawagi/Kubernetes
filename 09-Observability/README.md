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

