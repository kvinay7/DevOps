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

# ☸ **KUBERNETES**

### **Q6: What is Kubernetes?**

Kubernetes is a **container orchestration platform** that **manages containers automatically**.

| Why Kubernetes is Needed | Example                       |
| ------------------------ | ----------------------------- |
| Auto restart on crash    | If pod fails, K8s restarts it |
| Scaling                  | Increase replicas             |
| Load balancing           | Distributes traffic           |
| Rolling updates          | New version rollout           |
| Self-healing             | Keeps service alive           |


### **Q7: Core Kubernetes Concepts**

| Concept        | Purpose                                        |
| -------------- | ---------------------------------------------- |
| **Pod**        | Smallest unit → runs one or more containers    |
| **Deployment** | Manages pods, allows scaling & rolling updates |
| **Service**    | Exposes Flask app to network                   |
| **ReplicaSet** | Ensures desired number of pods                 |
| **Node**       | Machine where pods run                         |
| **kubectl**    | CLI to interact with cluster                   |


### **Q8: Kubernetes Deployment (Example)**

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


### **Q9: Kubernetes Service (Expose App)**

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


### **Q10: Kubernetes Commands**

```bash
kubectl apply -f k8s/deployment.yaml
kubectl get pods
kubectl get svc
kubectl logs <pod-name>
kubectl rollout restart deployment/flask-deploy
```

---

# ⚙ **GITHUB ACTIONS**

### **Q11: What is GitHub Actions?**

GitHub Actions is a **CI/CD automation tool** built into GitHub.
It runs tasks **automatically** when events occur (push, pull request, etc).


### **Q12: Core Concepts**

| Concept             | Purpose                            |
| ------------------- | ---------------------------------- |
| **Workflow**        | CI/CD pipeline                     |
| **Job**             | A set of steps                     |
| **Step**            | Individual command                 |
| **Runner**          | Machine that runs the job          |
| **Trigger (`on:`)** | Decide when workflow runs          |
| **Secrets**         | Sensitive data (tokens, passwords) |


### **Q13: CI Example – Build & Push Docker Image**

`.github/workflows/ci.yml`

```yaml
name: CI - Build & Push Docker Image

on:
  workflow_dispatch:   # ← allows manual run
  # push:
  #   branches: ["main"]

jobs:
  build-push:
    runs-on: ubuntu-latest    # includes Docker, so no need to install it manually.
    steps:
    - uses: actions/checkout@v3

    - name: Build Docker image
      run: docker build -t flask-app .

    - name: Login & Push
      run: |
        echo "${{ secrets.DOCKERHUB_PASSWORD }}" |
        docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin
        docker tag flask-app ${{ secrets.DOCKERHUB_USERNAME }}/flask-app:latest
        docker push ${{ secrets.DOCKERHUB_USERNAME }}/flask-app:latest
```

### **Q14: CD Example — Deploy to Kubernetes**

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

---

# **Conclusion:**

> “Built an end-to-end CI/CD pipeline using Docker, GitHub Actions, and Kubernetes.
> On manual trigger, GitHub Actions automatically builds and pushes a Docker image to Docker Hub,
> and a second workflow updates the Kubernetes manifest and deploys it live.”
