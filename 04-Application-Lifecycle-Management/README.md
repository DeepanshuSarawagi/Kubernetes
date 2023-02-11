# Application Lifecycle Management in Kubernetes

## Table of contents:

1. [Rollout and Rollback](#1-rolling-updates-and-rollbacks-)


## 1. Rolling updates and Rollbacks:

We can check the status of rolling updates to a kubernetes deployment by using the following command - 
```kubectl rollout status deployment/DEPLOYMENT_NAME```

To see the revisions and history of rollouts -
```kubectl rollout history deployment/DEPLOYMENT_NAME```

### 1a. Deployment Strategy:

Recreate

: In this deployment strategy, all the deployment pods are killed/terminated first and then the new pods are created. This
is not a recommended strategy since application will go down.

RollingUpdate

: In this strategy, we do not destroy the pods all at once. Instead, pods are killed/terminated one by one and a new version
of pod is created in rolling manner. This is the default deployment strategy.

To apply changes to a pod, we use the ```kubectl apply -f deployment.yaml``` command to apply new changes to PODs. Refer
[deployment.yaml](../Deployments/deployment.yaml) to inspect the deployment strategy field.

Kubernetes deployments, allow you to rollback an update if there is an issue with newer version. We can use following command
```kubectl rollout undo deployment/DEPLOYMENT_NAME```. Kubernetes will destroy the pods in new replicas set and will bring the
pods up in old replica set.