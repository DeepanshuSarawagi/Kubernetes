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