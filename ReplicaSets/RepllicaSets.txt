# Brief document on Replica Sets or Replication Controller

- For high availability, we need to have multiple instances of alike Pods running in a Kubernetes cluster.
- Replication controller ensures specified number of pods are running at the same time.
- It is also used to share the load across them i.e., load balancing among pods.
- Replica set is a newer technology and replaces replica controller.
- We need to use the pod template inside the spec section of replication controller definition yaml.
- replicas is the child element of spec section when creating replication controller using yaml.
- The main difference between replication controller and replicasets is that, we need to specify "selector" element
  in spec section.
- The selector definition helps the replica set identify what pods fall under it.
- The reason why need to specify selector definition is that it can also manage pods that were not created as part of the
  replica set creation.
- We need to specify the matchLabels which is a child element of selector definition
- The matchLabel will match all the labels with the specified type as defined in the yaml.
- If we specify the replicas to be 5, then replicaset will create 5 replica pods.
- In case if we manually create a new pod with matching label as specified in the template, replicaset will
  delete the 6th (additional) pod which we created manually.

Commands:
 - Command to apply/create replica sets using kubectl
   - $kubectl create -f ReplicaSets\rc-definition.yaml
 - Command to show replica sets
   - $kubectl get replicasets
 - Command to describe replica sets
   - $kubectl describe replicasets myapp-rc
 - Command to view wide info about replica sets
   - $kubectl describe replicasets myapp-rc
 - Command to check which api version ReplicaSet use
   - $kubectl explain replicasets
