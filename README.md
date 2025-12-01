<h1 align="center">DevOps - CI/CD</h1>

## 🐳 **DOCKER**

### **Q1: What is Docker?**

Docker is a **containerization platform** — it packages an application **with dependencies, libraries & runtime** into one unit called a **container**, which runs **same on any machine**.


### **Q2: What are Docker Core Concepts?**

| Concept          | Explanation                                          |
| ---------------- | ---------------------------------------------------- |
| **Image**        | Template/blueprint (code + dependencies)             |
| **Container**    | Running instance of an image                         |
| **Dockerfile**   | Instructions to build an image                       |
| **Registry**     | Storage for images (Docker Hub, ECR)                 |
| **Layer**        | Docker builds images in layers → faster builds       |
| **Volume**       | Stores data outside container (e.g. DB)              |
| **Port Mapping** | Allows external users to access app (`-p 5000:5000`) |
| **Tag**          | Version of image (`:v1`, `:latest`)                  |



### **Q3: How to create Dockerfile?**

```dockerfile
FROM python:3.9-slim         # Base OS + Python
WORKDIR /app                 # Folder inside container
COPY requirements.txt .      # Copy dependency file
RUN pip install -r requirements.txt   # Install dependencies
COPY . .                     # Copy source code
CMD ["python", "app.py"]     # Start the app
```

| Line                | Purpose                                 |
| ------------------- | --------------------------------------- |
| `FROM`              | Which base environment to use           |
| `WORKDIR`           | Default folder inside container         |
| `COPY`              | Copies files from local to container    |
| `RUN`               | Runs installation commands              |
| `CMD`               | Command that runs when container starts |
| `EXPOSE` (optional) | Tells which port app runs on            |



### **Q4: How to test Docker locally?**

```bash
docker build -t myapp:v1 .
docker run -d -p 5000:5000 myapp:v1
curl http://localhost:5000
docker ps
```



### **Q5: Push to Docker Hub**

```bash
docker login
docker tag myapp:v1 username/myapp:v1
docker push username/myapp:v1
```

---

# ⚙ **GITHUB ACTIONS**

### **Q1: What is GitHub Actions?**

GitHub Actions is a **CI/CD automation tool** built into GitHub.
It runs tasks **automatically** when events occur (push, pull request, etc).


### **Q2: Core Concepts**

| Concept             | Purpose                            |
| ------------------- | ---------------------------------- |
| **Workflow**        | CI/CD pipeline                     |
| **Job**             | A set of steps                     |
| **Step**            | Individual command                 |
| **Runner**          | Machine that runs the job          |
| **Trigger (`on:`)** | Decide when workflow runs          |
| **Secrets**         | Sensitive data (tokens, passwords) |


### **Q3: CI Example – Build & Push Docker Image**

`.github/workflows/ci.yml`

```yaml
name: CI - Build & Push Docker Image

# Trigger the workflow manually or on push to the main branch
on:
  workflow_dispatch:   # ← allows manual run
  # Uncomment this to trigger the workflow on push to 'main'
  # push:
  #   branches: ["main"]

# Jobs defines the set of tasks to run.
jobs:
  build-push:
    runs-on: ubuntu-latest  # includes Docker, so no need to install it manually

    steps:
    # Checkout the code from the repository
    - name: Checkout code
      uses: actions/checkout@v3  # Uses the latest version of the checkout action

    # Build the Docker image
    - name: Build Docker image
      run: docker build -t flask-app .  # Build image with the tag 'flask-app'

    # Docker login, tagging, and pushing to DockerHub
    - name: Login to DockerHub and Push
      run: |
        # Login to DockerHub using the credentials stored in GitHub secrets
        echo "${{ secrets.DOCKERHUB_PASSWORD }}" | docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin

        # Tag the built image with your DockerHub repository name
        docker tag flask-app ${{ secrets.DOCKERHUB_USERNAME }}/flask-app:latest

        # Push the image to DockerHub
        docker push ${{ secrets.DOCKERHUB_USERNAME }}/flask-app:latest
```

### DockerHub Authentication:
To authenticate Docker to push to DockerHub, need to add DockerHub credentials (username and password/token) in the GitHub repository's secrets.

To generate an access token in DockerHub:

1. Log into [DockerHub](https://hub.docker.com/).
2. Go to **Account Settings** > **Security**.
3. Under **New Access Token**, create a token with `read` and `write` permissions.
4. Copy the token and use it as `DOCKERHUB_PASSWORD` in GitHub secrets.
   
### DockerHub Credentials in GitHub Secrets:

   * GitHub Secrets are encrypted environment variables used to store sensitive data like passwords or tokens.
   * Go to your GitHub repository, navigate to **Settings > Secrets and variables > Actions**, and then add:

     * `DOCKERHUB_USERNAME`: DockerHub username.
     * `DOCKERHUB_PASSWORD`: DockerHub password or **DockerHub Access Token**.
  
### Running the CI Workflow:

* Manually trigger the workflow by going to **Actions** > **CI - Build & Push Docker Image** > **Run Workflow** in GitHub repo.
* Ensure that **Dockerfile** and all necessary application files (e.g., `requirements.txt`, `app.py`, etc.) are present in the repository.

---

# ☸ **KUBERNETES**

### **Q1: What is Kubernetes?**

Kubernetes is a **container orchestration platform** that **manages containers automatically**.

| Why Kubernetes is Needed | Example                       |
| ------------------------ | ----------------------------- |
| Auto restart on crash    | If pod fails, K8s restarts it |
| Scaling                  | Increase replicas             |
| Load balancing           | Distributes traffic           |
| Rolling updates          | New version rollout           |
| Self-healing             | Keeps service alive           |


### **Q2: Core Kubernetes Concepts**

| Concept        | Purpose                                        |
| -------------- | ---------------------------------------------- |
| **Pod**        | Smallest unit → runs one or more containers    |
| **Deployment** | Manages pods, allows scaling & rolling updates |
| **Service**    | Exposes Flask app to network                   |
| **ReplicaSet** | Ensures desired number of pods                 |
| **Node**       | Machine where pods run                         |
| **kubectl**    | CLI to interact with cluster                   |


### **Q3: Kubernetes Deployment (Example)**

`k8s/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-deploy
spec:
  replicas: 1
  selector:
    matchLabels: { app: flask }
  template:
    metadata:
      labels: { app: flask }
    spec:
      containers:
      - name: flask-container
        image: kvinay7/flask-app:latest
        ports:
        - containerPort: 5000
```


### **Q4: Kubernetes Service (Expose App)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: LoadBalancer
  selector:
    app: flask
  ports:
    - port: 80
      targetPort: 5000
```


### **Q5: Kubernetes Commands**

```bash
kubectl apply -f k8s/deployment.yaml
kubectl get pods
kubectl get svc
kubectl logs <pod-name>
kubectl rollout restart deployment/flask-deploy
```


### **Q6: CD Example — Deploy to Kubernetes**

`.github/workflows/cd.yml`

```yaml
name: CD - Deploy to Kubernetes

on:
  workflow_run:
    workflows: ["CI - Build & Push Docker Image"]
    types: [completed]

jobs:
  deploy:
    runs-on: self-hosted   # Codespace runner or Cloud K8s

    steps:
    - uses: actions/checkout@v3

    - name: Update image
      run: |
        sed -i "s|image:.*|image: ${{ secrets.DOCKERHUB_USERNAME }}/flask-app:latest|g" k8s/deployment.yaml

    - name: Deploy
      run: |
        kubectl apply -f k8s/deployment.yaml
        kubectl rollout restart deployment/flask-deploy
```

### **1. Self-Hosted Runner Requirements**

* `kubectl` installed
* `sed` installed (usually default on Linux)
* Access to Kubernetes cluster (via `~/.kube/config` or exported `KUBECONFIG`)
* Permission to run:

  * `kubectl apply`
  * `kubectl rollout restart`

---

### **2. Kubernetes Cluster Requirements**

* `k8s/deployment.yaml` (app deployment)
* `k8s/service.yaml` (**required** to expose app)
* Optional: `k8s/ingress.yaml` (if using domain name)

---

### **3. Kubernetes Access Type → How Get the Live URL**

#### **Option A: LoadBalancer Service**

* Create a `Service` with `type: LoadBalancer`
* Run:

  ```
  kubectl get svc flask-service
  ```
* EXTERNAL-IP becomes **live URL**

#### **Option B: Ingress (recommended)**

* Requires an ingress controller (e.g., NGINX)
* Create `ingress.yaml`
* DNS points to ingress controller
* App becomes available
