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

# Part 4: Configuration & Secrets

## Objective

Enhance the Nginx deployment by using a ConfigMap for application configuration and a Secret for storing credentials.

---

## Create ConfigMap

### configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: dev
```

Apply the ConfigMap:

```bash
kubectl apply -f configmap.yaml
```

Verify:

```bash
kubectl get configmap
```

Example Output:

```text
NAME         DATA   AGE
app-config   1      1m
```

---

## Create Secret

### secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  USERNAME: admin
  PASSWORD: admin123
```

Apply the Secret:

```bash
kubectl apply -f secret.yaml
```

Verify:

```bash
kubectl get secrets
```

Example Output:

```text
NAME          TYPE     DATA   AGE
app-secret    Opaque   2      1m
```

---

## Update Deployment

The existing Nginx Deployment was updated to consume values from the ConfigMap and Secret as environment variables.

### Environment Variable Configuration

```yaml
env:
- name: APP_MODE
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_MODE

- name: USERNAME
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: USERNAME

- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: PASSWORD
```

Apply the updated deployment:

```bash
kubectl apply -f deployment.yaml
```

Output:

```text
deployment.apps/nginx-deployment unchanged
```

---

## Verify Environment Variables Inside Container

List running Pods:

```bash
kubectl get pods
```

Example Output:

```text
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5c4ff99745-29rhh   1/1     Running   0          66s
nginx-deployment-5c4ff99745-hl4tz   1/1     Running   0          63s
```

Verify ConfigMap value:

```bash
kubectl exec -it nginx-deployment-5c4ff99745-29rhh -- printenv APP_MODE
```

Output:

```text
dev
```

Verify Secret username:

```bash
kubectl exec -it nginx-deployment-5c4ff99745-29rhh -- printenv USERNAME
```

Output:

```text
admin
```

Verify Secret password:

```bash
kubectl exec -it nginx-deployment-5c4ff99745-29rhh -- printenv PASSWORD
```

Output:

```text
admin123
```

---

## Observation

A ConfigMap named `app-config` was used to store non-sensitive application settings, while a Secret named `app-secret` was used to store credentials securely.

The Deployment successfully injected both ConfigMap and Secret values into the container as environment variables.

Verification using `kubectl exec` confirmed that the application received the expected values (`APP_MODE=dev`, `USERNAME=admin`, and `PASSWORD=admin123`).

## Screenshoot:
1. <img width="883" height="140" alt="Screenshot_4" src="https://github.com/user-attachments/assets/03d66c0c-0343-4b22-b9d1-9a10c25f3c61" />
2. <img width="900" height="117" alt="Screenshot_5" src="https://github.com/user-attachments/assets/f1493d78-1f1f-4afe-9f65-6a03c63fdc8b" />
3. <img width="886" height="82" alt="Screenshot_6" src="https://github.com/user-attachments/assets/2baedcbc-465a-474e-88aa-a397ec027d1e" />
4. <img width="924" height="257" alt="Screenshot_7" src="https://github.com/user-attachments/assets/a5b43f81-238e-4956-bc46-2002b9838d9e" />

# Part 5: Scaling & Rolling Updates

## Objective

Scale the Nginx Deployment and perform a rolling update followed by a rollback.

---

## Scaling the Deployment

The Deployment was scaled from 2 replicas to 4 replicas.

### Command

```bash
kubectl scale deployment nginx-deployment --replicas=4
```

### Output

```text
deployment.apps/nginx-deployment scaled
```

### Verification

```bash
kubectl get deployment nginx-deployment
```

Example Output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   4/4     4            4
```

```bash
kubectl get pods
```

Example Output:

```text
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-xxxxx              1/1     Running   0
nginx-deployment-xxxxx              1/1     Running   0
nginx-deployment-xxxxx              1/1     Running   0
nginx-deployment-xxxxx              1/1     Running   0
```

---

## Rolling Update

A rolling update was performed by changing the Nginx image version.

### Command

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.27
```

### Output

```text
deployment.apps/nginx-deployment image updated
```

### Verify Rollout Status

```bash
kubectl rollout status deployment/nginx-deployment
```

Output:

```text
deployment "nginx-deployment" successfully rolled out
```

### Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

Output:

```text
deployment.apps/nginx-deployment

REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

---

## Rollback

The deployment was rolled back to the previous version.

### Command

```bash
kubectl rollout undo deployment/nginx-deployment
```

### Output

```text
deployment.apps/nginx-deployment rolled back
```

### Verify Rollback

```bash
kubectl rollout status deployment/nginx-deployment
```

Output:

```text
deployment "nginx-deployment" successfully rolled out
```

---

## Observation

The Deployment was successfully scaled from 2 replicas to 4 replicas, and Kubernetes automatically created additional Pods to maintain the desired state.

A rolling update was performed by updating the Nginx image version. Kubernetes gradually replaced the old Pods with new ones while keeping the application available during the update process.

The rollback operation restored the previous stable version of the Deployment. This demonstrates Kubernetes' ability to perform safe updates and quickly recover from configuration changes when needed.

## Screenshoot:
1. <img width="924" height="257" alt="Screenshot_7" src="https://github.com/user-attachments/assets/0f85d8a9-f998-4bfb-9691-336cb4dbd0c3" />
2. <img width="928" height="100" alt="Screenshot_8" src="https://github.com/user-attachments/assets/250f3e57-4912-4bed-919d-cc2f3b5d242b" />
3. <img width="914" height="171" alt="Screenshot_9" src="https://github.com/user-attachments/assets/438083a2-7478-466c-acb1-533509705d40" />
4. <img width="909" height="118" alt="Screenshot_10" src="https://github.com/user-attachments/assets/637cd6d3-c56d-4200-bd68-aeccd4fae8c6" />
5. <img width="915" height="79" alt="Screenshot_11" src="https://github.com/user-attachments/assets/7d241f9c-df02-4ec0-a3d2-00d342ffbf0a" />
6. <img width="907" height="166" alt="Screenshot_12" src="https://github.com/user-attachments/assets/55e54096-bdf2-4a9e-9dc8-86f29511b835" />
7. <img width="1027" height="188" alt="Screenshot_13" src="https://github.com/user-attachments/assets/a23eff73-ccf9-4f2d-9a26-c0699b1f7f76" />
8. <img width="897" height="78" alt="Screenshot_14" src="https://github.com/user-attachments/assets/560c9d00-5f7f-4917-87b0-e38a32b61606" /> 












