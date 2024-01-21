# Jobs and CronJobs in Kubernetes


## Table of contents:

1. [Introduction](#1-introduction)


## 1. Introduction:

There are workloads that are meant to live for a short period of time. For example, performing a math operation, creating
reports, sending an email report.

Let's see how we can achieve this using Docker.

```shell
$docker run ubunty expr 3 + 2
```
The docker container will perform the express and move into exited state.

Now let's see how we can achieve this using Kubernetes.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: math-job-pod
spec:
  containers:
    - name: math-job-pod
      image: ubuntu
      imagePullPolicy: IfNotPresent
      command:
        - 'expr'
        - '3'
        -  '+'
        - '2'
      restartPolicy: Always        #  Change this to Never
```

The Kubernetes POD will perform the computational task, however, the POD will continue to run. Since, it wants the application
to run forever. This is because of the ```restartPolicy``` which is set to ```Always``` by default.

## 2. Jobs: