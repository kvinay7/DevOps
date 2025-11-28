## Harness CI/CD

**Goal - Whenever we push to GitHub → Harness should automatically:**

1. Pull code
2. Build and Push Docker image to container registry
3. Pull and Deploy image to K8s Cluster

---

### Step 1 – Create Docker Hub Repo

1. Go to Docker Hub.
2. Create Repository → e.g. `yourname/flask-harness-demo`.
3. Keep it **public** (easier for free).

---

### Step 2 – Add GitHub Connector in Harness

In Harness UI:

1. Go to **Project → Connectors**.
2. Click **New Connector → Git**.
3. Choose **GitHub**.
4. Provide:

   * Repo URL: `https://github.com/yourname/flask-harness-demo.git`
   * Auth:

     * PAT (Personal Access Token) or OAuth (if org policy allows)
5. Click **Test & Save**.

Now Harness can read code.

---

### Step 3 – Add Docker Hub Connector (Artifact / Registry)

1. In Harness, go to **Connectors → New Connector → Docker Registry**.
2. Set:

   * **URL**: `https://index.docker.io/v1/`
   * **Username**: your Docker Hub username
   * **Password / Token**: Docker Hub access token
3. **Test & Save**.

Now Harness can push images to Docker Hub.

---

### Step 4 – Create a CI Pipeline in Harness

In your Harness project:

1. Go to **Pipelines → New Pipeline**.
2. Name it: `flask-ci-pipeline`.
3. Choose **CI** as pipeline type (Build).

Inside pipeline:

#### ✅ Add a Stage – “Build & Push Docker Image”

1. Click **Add Stage → Build → Name: build-docker**.

2. **Infrastructure**:

   * Use **Kubernetes cluster** (if org has one)
   * Or use **Hosted Harness Build Infra** if available.

3. Inside Stage → Add a **Step**:

   * Type: **Run** (or **Build & Push Docker** if available in your template)

If using a shell **Run Step**, commands would roughly be:

```bash
# Login to Docker Hub
docker login -u <+secrets.getValue("docker_username")> -p <+secrets.getValue("docker_password")>

# Build image
docker build -t yourname/flask-harness-demo:${HARNESS_BUILD_ID} .

# Tag & push
docker tag yourname/flask-harness-demo:${HARNESS_BUILD_ID} yourname/flask-harness-demo:latest
docker push yourname/flask-harness-demo:${HARNESS_BUILD_ID}
docker push yourname/flask-harness-demo:latest
```

You’d store `docker_username` and `docker_password` as **secrets** in Harness.

> The exact UI fields may be different, but conceptually:
> **Step = Build Docker Image + Push to Docker Hub**.

---

### Step 5 – Configure Pipeline Trigger (On Git Push)

1. Inside the pipeline → go to **Triggers**.
2. **New Trigger → GitHub Webhook / Push**.  
3. Choose:

   * Connector: your GitHub connector.
   * Event: `Push` on branch `main` or `dev`.
4. Save.

Now:
**Whenever you push code → Harness CI pipeline runs → builds & pushes Docker image to Docker Hub** ✅

---

### Step 7 – CD Stage in Harness

