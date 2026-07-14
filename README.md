## module-11-kubernetes-assignment
# Part 1: Conceptual Understanding
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

# Part 3: Multi-Resource Deployment

## Objective

Deploy an Nginx application in Kubernetes using a Deployment and expose it using a NodePort Service.

---

## Deployment Manifest

Created a Deployment with 2 replicas using Nginx.

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

## Service Manifest

Created a NodePort Service to expose the application externally.

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-app
  ports:
  - port: 80
    targetPort: 80
```

---

## Apply Resources

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Output:

```text
deployment.apps/nginx-deployment created
service/nginx-service created
```

---

## Verification

### Check Pods

```bash
kubectl get pods
```

Output:

```text
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-667f7bdcb9-8q64d   1/1     Running   0
nginx-deployment-667f7bdcb9-nfcjt   1/1     Running   0
```

### Check Deployment

```bash
kubectl get deployments
```

Output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2
```

### Check Service

```bash
kubectl get services
```

Output:

```text
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)
nginx-service   NodePort    10.109.46.22   <none>        80:31732/TCP
```

---

## Access the Application

Generate the service URL:

```bash
minikube service nginx-service --url
```

Open the generated URL in a web browser.

### Result

The default **Nginx Welcome Page** was displayed successfully.

---

## Observation

The Deployment successfully created and maintained two Nginx Pods as specified in the replica configuration.

The NodePort Service exposed the application and routed incoming traffic to the running Pods.

Accessing the service URL in the browser displayed the default Nginx welcome page, confirming that the application was deployed and running successfully.

## Screenshoot:
1. <img width="1002" height="883" alt="Screenshot_2" src="https://github.com/user-attachments/assets/d79b0e8b-f003-450c-bb41-a86ac0ccf0cc" />
2. <img width="1385" height="781" alt="Screenshot_3" src="https://github.com/user-attachments/assets/8f1ac97a-7b2f-4be8-b6a9-b5931ca4c432" />


