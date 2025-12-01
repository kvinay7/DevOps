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
    runs-on: ubuntu-latest
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
