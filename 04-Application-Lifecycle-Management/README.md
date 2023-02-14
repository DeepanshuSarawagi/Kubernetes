# Application Lifecycle Management in Kubernetes

## Table of contents:

1. [Rollout and Rollback](#1-rolling-updates-and-rollbacks-)
   1. [Deployment Strategy](#1a-deployment-strategy-)
2. [Application Commands](#2-application-commands-)
3. [Environment Variables](#3-environment-variables-)
4. [Config Maps](#4-config-maps-)
   1. [Imperative Way](#4a-imperative-way-to-create-cm-)
   2. [Declarative Way](#4b-declarative-way-to-create-cm-)
   3. [Inject cm into PODs](#4c-inject-configmpas-into-pods-)


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

## 2. Application Commands:

By default, docker does not attach a terminal to a container when it is run. To have a container running as long as we want,
we need to pass commands and arguments in the CMD arg of Docker file. The commands and arguments should be a JSON-array. 
The first command should always be a valid executable.

[ENTRYPOINT] is the command to which you can append arguments in the docker run command.

If we specify both [CMD] and [ENTRYPOINT] then CMD instruction would be appended with ENTRYPOINT instruction.

When it comes to pod definition, if we want to specify any argument which docker run commands expects, it can go into the args
of the spec section in [pod-definition.yaml](pod-args.yaml). 

## 3. Environment Variables:

We can configure environment variables required by pods using env field under spec section of [pod-env.yaml](pod-env.yaml)

## 4. Config maps:

We can manage the environment variables required for multiple pods centrally using config maps. Config maps are used to pass
configuration data in the form of key-value pair.

### 4a. Imperative way to create cm:

We can use following kubectl command to create config maps imperatively.

```kubectl create configmap app_config --from-literal=APP_COLOR=Blue --from-literal=APP_MODE=Dev```

Another way to create configmaps is using file. We can pass the file with required data to create config maps.

```kubectl create configmap app_config --from-file=app_config.properties```

### 4b. Declarative way to create cm:

We can use the [config-maps.yaml](config-maps.yaml) to create configmaps. And then use the kubectl command to create the
configmaps

```kubectl create -f config-maps.yaml```

### 4c. Inject configmpas into PODs:

Once the configmaps are create, we need to inject them into pods. A sample pod definition yaml can be found [here](pods-config-maps.yaml).
