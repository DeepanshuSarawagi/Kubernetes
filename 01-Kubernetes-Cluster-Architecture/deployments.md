# This document will give brief explanation on how deployments work in Kubernetes

- The Deployment object in Kubernetes provides us the capability to upgrade the underlying instances seamlessly using the
  rolling updates.
- Use the below command to create deployment
  - $kubectl create -f deployment.yaml --record
  - --record option helps you record and show changes when you run the rollout history command

Rollouts and Versioning:
- When you first create a deployment, it triggers a rollout.
- Rollout creates a new deployment version 1.
- When a deployment definition is updated, a new rollout is triggered with version 2.
- Use the below command to apply the updates:
  - $kubectl apply -f deployment-definition.yaml
- if you want to update the deployment on the fly without editing the yaml file, below is the command
  - $kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v2
    - Here, simple-webapp is the container name. You need to specify the container name whose image needs to be updated.

- To view the status of the rollout below is the command:
  - $kubectl rollout status deployment/httpd-frontend
- To view the history of the rollouts, below is the command:
  - $kubectl rollout history deployment/httpd-frontend
- To view all deployment related objects below is the command:
  - $kubectl get all

Deployment strategies:
- One of rolling out the new updates to the deployment is by deleting all the pods and then creating the new ones.
- This isn't the best deployment strategy.
- This is called as Recreate strategy.
- Next deployment strategy is the RollingUpdate strategy which kills the pod one by one and spins up updated pods one by one.
- This way application availability isn't affected.
- This is the default strategy.

Rollback:
- We can undo the deployment or rollback the update by using the below command:
  - $kubectl rollout undo deployment/httpd-frontend

Exam tip:
- If we are asked to create a deployment, it may be difficult to create YAML file during exam, hence, we can use the
  ```$kubectl run``` command to run a pod with specified image and replicas set.

## Useful commands to generate yamls and create pods

```shell
#Create an NGINX Pod
$kubectl run nginx --image=nginx

#Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

$kubectl run nginx --image=nginx --dry-run=client -o yaml

#Create a deployment

$kubectl create deployment --image=nginx nginx

#Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

$kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

#Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

$kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

#Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.

$kubectl create -f nginx-deployment.yaml

#OR

#In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.

$kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml

# To check the status of Deployment, run the following command

$kubectl rollout status deployment/myapp-deployment

# To check the history of deployments

$kubectl rollout history deployment/myapp-deployment

# To record a deployment

$kubectl create deployment -f deployment.app --record


```

