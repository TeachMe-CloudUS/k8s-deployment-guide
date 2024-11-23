# **Deploying Your Service to the Kubernetes Cluster in Azure**

This guide provides all the steps you need to deploy your service to our shared Kubernetes cluster in Azure and interact with other services within the cluster.

---

## **Prerequisites**
Before you begin, make sure you have the following tools installed locally:
1. **kubectl**: CLI tool for Kubernetes.
    - Installation guide: [kubectl Install Guide](https://kubernetes.io/docs/tasks/tools/#kubectl)
2. **Azure CLI**: CLI tool to interact with Azure.
    - Installation guide: [Azure CLI Install Guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
3. **Docker**: To build container images for your service.
    - Installation guide: [Docker Install Guide](https://docs.docker.com/get-docker/)
4. **Access to our Azure Kubernetes Service (AKS)**:
    - Make sure you’ve been added to the Azure project and have access to the AKS cluster.

---

## **Step 1: Clone This Repository**
Clone this repository, which contains all the necessary files for deploying your service:

```bash
git clone https://github.com/our-org/k8s-deployment-guide.git
cd k8s-deployment-guide
```

## Step 2: Prepare Your Service for Deployment
Your service needs to meet the following requirements:

### 1. Dockerized Service
Your service should have a Dockerfile in the root directory.
Build and push your Docker image to a container registry (e.g., Azure Container Registry (ACR) or DockerHub).

**Steps to Push to a Registry:**

1. Build the Docker Image:
```bash
docker build -t <your-registry>/<your-service-name>:<version> .
```

2. **Push the Docker Image:**

```bash
docker push <your-registry>/<your-service-name>:<version>
```

### 2. Kubernetes Deployment Files
-  Copy the template files from the `k8s-manifests-templates` folder in this repository:
   - `deployment.yaml`
   - `service.yaml`
   - `configmap.yaml`
   - `secrets.yaml`
- Update these files with your service details (see **Step 3**).

## **Step 3: Configure Your Kubernetes Deployment**

1. Edit `deployment.yaml`:
Replace placeholders in the `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
   name: <your-service-name>
spec:
   replicas: 2
   selector:
      matchLabels:
         app: <your-service-name>
   template:
      metadata:
         labels:
            app: <your-service-name>
      spec:
         containers:
            - name: <your-service-name>
              image: <your-registry>/<your-service-name>:<version>
              ports:
                 - containerPort: 8080
```

2. Edit `service.yaml`:
Define a Kubernetes service to expose your deployment:

```yaml
apiVersion: v1
kind: Service
metadata:
   name: <your-service-name>
spec:
   selector:
      app: <your-service-name>
   ports:
      - protocol: TCP
        port: 80
        targetPort: 8080
   type: ClusterIP
```

## **Step 4: Manage Environment Variables Using Secrets and ConfigMaps**

### 1. ConfigMaps (Non-Sensitive Data)

Use ConfigMaps to store non-sensitive configuration data like API endpoints or feature flags.

**Create a ConfigMap**

1. Define your ConfigMap in `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
   name: my-configmap
data:
   FEATURE_FLAG: "true"
   API_ENDPOINT: "https://api.example.com"
```

2. Apply the ConfigMap:

```bash
kubectl apply -f configmap.yaml
```

**Use the ConfigMap in Your Deployment**

Update your `deployment.yaml`:

```yaml
env:
   - name: FEATURE_FLAG
     valueFrom:
        configMapKeyRef:
           name: my-configmap
           key: FEATURE_FLAG
   - name: API_ENDPOINT
     valueFrom:
        configMapKeyRef:
           name: my-configmap
           key: API_ENDPOINT
```

### 2. Secrets (Sensitive Data)
Use Secrets to store sensitive data like passwords, API keys, or database credentials.

**Create a Secret**

1. Define your Secret in `secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
   name: my-secret
type: Opaque
data:
   sensitive-key: dXNlcm5hbWU=  # Base64-encoded value
```

- To encode a value to Base64:
```bash
echo -n "your-sensitive-value" | base64
```

**It is important to store the secret data in Base64!**

2. Apply the Secret:
```bash
kubectl apply -f secret.yaml
```

**Use the Secret in Your Deployment**

Update your `deployment.yaml`:

```yaml
env:
   - name: SENSITIVE_ENV
     valueFrom:
        secretKeyRef:
           name: my-secret
           key: sensitive-key
```

**Never store the `secret.yaml` in the repository!!**<br>
**Just add `secret.yaml` and `configmap.yaml` to your projects `.gitignore`.**
<br></br>

To see a working example of this configuration just see [teachme-student-service](https://github.com/TeachMe-CloudUS/teachme-student-service/tree/k8/k8s-manifests).


## **Step 5: Deploy Your Service to the Cluster**

1. Connect to the AKS Cluster: Log in to Azure and configure access to the Kubernetes cluster:
```bash
az login
az aks get-credentials --resource-group fis2425-rgp --name teachme-k8s-cluster
```

2. Deploy Your Service: Apply the `deployment.yaml` and `service.yaml` files:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

3. **Verify Deployment:** Check if your service is running:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## **Step 6: Accessing Your Service**

### For local testing: Port-forwarding
You can access your service locally using `kubectl port-forward`.
This is useful for debugging or testing without exposing your service externally.

1. Find the name of your pod:
```bash
kubectl get pods
```

2. Forward a local port to the pod:
```bash
kubectl port-forward <pod-name> 8080:8080
```

3. Access the service at `http://localhost:8080`.

### Inter-Service Communication
In the cluster, services can communicate with each other using their **service names**.
- Use the `<service-name>` defined in `service.yaml` file.
-  Example: If you want to call the `course-service`, use:
   - `http://course-service`
     
## **Step 7: Updating your service**
When you have updated your service you have to repeat the following steps:

1. Build and push a new version of your Docker image:
2. Update your deployment to use the new image:
```bash
kubectl set image deployment/<your-service-name> <your-service-name>=<your-registry>/<your-service-name>:<new-version>
```
3. Verify the update:
```bash
kubectl rollout status deployment/<your-service-name>
```

## Github Workflows

If you want to do Continues Deployment, have a look at the [k8-deployment.yml](https://github.com/TeachMe-CloudUS/teachme-student-service/blob/k8/.github/workflows/k8-deployment.yml)
of the `teachme-student-service`.
