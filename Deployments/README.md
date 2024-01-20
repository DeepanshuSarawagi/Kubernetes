# Deployment Strategies

## Table of contents:

1. [Introduction](#1-introduction)
2. [Blue/Green deployments](#2-bluegreen-deployment)

## 1. Introduction:
In this chapter, we are going to talk about the different deployment strategies. There are several deployment strategies
available such as 

- Recreate
- RollingUpdate

Apart from the conventional way of deploying pods using Kubernetes, we can also follow certain best practices. Two of such
best practices are

-  Canary Deployment
- Blue/Green Deployment

## 2. Blue/Green Deployment:

In this type of deployment, there are two version of deployments created. Lets say v1 or v2.

V1 can be called as Blue(existing) deployment. The kube service would be configured to route traffic to this version of
deployment.

V2 version of deployment is created which is called as Green Deployment. Once all the functionality is tested, we then update
the service to route traffic to Green Deployment.

Refer to the [blue-deployment.yaml](blue-deployment.yaml) and [green-deployment.yaml](green-deployment.yaml) for 
examples. If you notice in the [green-deployment.yaml](green-deployment.yaml), the same service forwards the traffic to v2
deployment.

## 3. Canary Deployments:

In Canary deployments, we can restrict very minimal traffic to version 2 of deployment, aka Canary Deployment. Once the tests are validated, we
can then delete the canary deployment and rollout the changes to existing PODs. We can achieve this by following kubernetes
primitives as below.

- Create a version1 of Deployment with 5 replicas.
- Create a service to route traffic to version1 deployment.
- Create a new canary deployment with updates and just 1 replica.
  - Ensure that the label of deployment is same as version1 deployment.
- Update the service's selector field to the common label between version1 and canary deployment.
  - This way traffic routes to both version1 and canary version of deployment.
  - 83% traffic would go to version1 and 17% traffic to canary.
- Once testing is complete, delete the canary deployment and rollout the changes to existing version1 deployment.

Refer to the following [canary-deployment.yaml](canary-deployment.yaml) file for example.