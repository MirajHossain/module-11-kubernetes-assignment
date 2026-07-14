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

# Part 2: Cluster Setup & Verification

## Step 1: Start the Cluster

bash:
minikube start


Result:
Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default.


## Step 2: Verify Cluster Context

bash:
kubectl config current-context


Output:
minikube

## Observation

The Minikube cluster was successfully created and configured. The current Kubernetes context is set to `minikube`, confirming that kubectl is connected to the cluster.

The cluster control plane started successfully using the Hyper-V driver and is ready for further verification and workload deployment.

## Screenshoot
<img width="986" height="956" alt="Screenshot_1" src="https://github.com/user-attachments/assets/80b1b957-60b4-47b0-81b8-55efc6ace41b" />

