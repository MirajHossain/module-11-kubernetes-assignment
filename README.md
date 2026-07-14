# module-11-kubernetes-assignment
1. How does Kubernetes ensure high availability compared to traditional deployment?
Kubernetes maintains the desired number of application instances using ReplicaSets and automatically restarts failed containers.
It distributes workloads across nodes and can reschedule Pods if a node fails.
Traditional deployments often require manual intervention for failover and scaling.
2. Explain the relationship between Pods, ReplicaSets, and Deployments.
A Pod is the smallest deployable unit that runs one or more containers.
A ReplicaSet ensures a specified number of identical Pod replicas are always running.
A Deployment manages ReplicaSets, providing features like rolling updates, rollbacks, and version control.
3. Why are Services required in Kubernetes?
Pods are ephemeral and their IP addresses can change when they restart.
A Service provides a stable network endpoint and DNS name for accessing Pods.
It also performs load balancing across multiple Pod instances.
4. Difference between ConfigMaps and Secrets with a practical example.
ConfigMaps store non-sensitive configuration data, while Secrets store sensitive information such as passwords or API keys.
Example: Store APP_ENV=production in a ConfigMap, and store a database password (DB_PASSWORD) in a Secret.
Applications can access both as environment variables or mounted files.
