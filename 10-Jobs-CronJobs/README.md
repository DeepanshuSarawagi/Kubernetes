# Jobs and CronJobs in Kubernetes


## Table of contents:

1. [Introduction](#1-introduction)
2. [Jobs](#2-jobs)


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

Consider a scenario when we require multiple pods to perform computation tasks such as performing report analysis and sending
an email. In this scenario, we want a manager that can manage the number of pods and ensure the work is done and the pods
exit once the work is done. Refer to [jobs.yaml](jobs.yaml) definition file for getting an idea on how it is different from ReplicaSet.

If a job was created to process an image, then we would need multicontainer pods. If we want 3-4 pods to perform jobs for
us, then we can set the ```completion``` field under spec. By default, the pods are created one after the other. 

Instead of getting the jobs completed sequentially, we can get the job completed parallel using the property called ```parallelism```
which will create the specified number of pods to complete the task.

## 3. CronJobs:

A CronJon in Kubernetes is like Crontab in Unix where you can schedule job to perform tasks. It accepts a Unix like string
to schedule jobs. More details about unix scheduling can be found in this [wiki](https://en.wikipedia.org/wiki/Cron) link.

refer to [cronjobs.yaml](cronjobs.yaml) for Kubernetes definition example.