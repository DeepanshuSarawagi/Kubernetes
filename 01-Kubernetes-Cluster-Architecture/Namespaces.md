# This document gives brief explanation on namespaces in Kubernetes

- A default namespace is created when cluster is first started.
- Kubernetes creates one more namespace called kube-system where pods for networking, controllers, api-servers are created
  which ensures to prevent users from accidentally deleting it.
- kube-public is where resources which should be made available to all users are created.
- You can create your own namespaces to isolate your environment.
- Each of these namespaces can have its own set of policies.
- When a service is created, a DNS record is automatically created. It is in the following format.
  - db-service.dev.svc.cluster.local
  - This is service-name.namespace.svc.domain