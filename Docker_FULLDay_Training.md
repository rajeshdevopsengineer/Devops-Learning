Absolutely. For a **full-day DevOps Docker training**, I recommend converting the introductory material into an **8-module, ~7.5-hour instructor-led workshop** with six labs. The emphasis should be roughly **35% concepts and 65% live demo/hands-on**.

# Full-Day Docker for DevOps — Trainer Package

**Audience:** DevOps Engineers, Cloud Engineers, System Administrators, developers moving into DevOps
**Level:** Beginner → Intermediate → Production-oriented
**Duration:** ~7.5 hours including breaks
**Environment:** Windows 10/11 + Docker Desktop + WSL2/Ubuntu + Azure subscription + Azure CLI + kubectl + Helm optional

## Training flow

| Time        | Module                              | Format            |
| ----------- | ----------------------------------- | ----------------- |
| 09:30–10:15 | Module 1 — Docker Fundamentals      | Theory + Demo     |
| 10:15–11:15 | Module 2 — Dockerfile Deep Dive     | Demo + Lab 1      |
| 11:15–11:30 | Break                               |                   |
| 11:30–12:15 | Module 3 — Docker Volumes           | Demo + Lab 2      |
| 12:15–13:00 | Module 4 — Docker Networking        | Demo + Lab 3      |
| 13:00–14:00 | Lunch                               |                   |
| 14:00–15:00 | Module 5 — Docker Compose           | Demo + Lab 4      |
| 15:00–15:45 | Module 6 — Azure Container Registry | Demo + Lab 5      |
| 15:45–16:00 | Break                               |                   |
| 16:00–16:40 | Module 7 — Docker Security          | Security workshop |
| 16:40–17:30 | Module 8 — Docker → AKS             | Lab 6             |
| 17:30–17:45 | Troubleshooting + Q&A               | Review            |

---

# Module 1 — Docker Fundamentals

### Trainer objective

Start with the problem Docker solves rather than starting with commands.

Explain:

```text
Traditional Deployment

Developer
    ↓
Application
    ↓
"Works on my machine"
    ↓
Different OS / runtime / dependencies
    ↓
Deployment Failure
```

Docker approach:

```text
Application
    +
Runtime
    +
Dependencies
    ↓
Docker Image
    ↓
--------------------------------
DEV     TEST     UAT     PROD
--------------------------------
        Same Artifact
```

Cover:

* Containers vs VMs
* Docker architecture
* Docker Engine
* Docker Desktop
* Images
* Containers
* Registries
* Container lifecycle
* Port mapping

### Live demo

Start with:

```bash
docker version
docker info
docker run hello-world
docker images
docker ps
docker ps -a
```

Then NGINX:

```bash
docker pull nginx

docker run -d \
  --name webserver \
  -p 8080:80 \
  nginx
```

Browser:

```text
localhost:8080
```

Continue:

```bash
docker logs webserver
docker inspect webserver
docker stats webserver
docker exec -it webserver bash
```

Trainer question:

> If I delete this container, what happens to the NGINX image?

Use the answer to reinforce **image ≠ container**.

---

# Module 2 — Dockerfile Deep Dive

This module should go considerably beyond the first Dockerfile.

## 2.1 Dockerfile instructions

Explain:

```dockerfile
FROM
WORKDIR
COPY
ADD
RUN
ENV
ARG
EXPOSE
USER
CMD
ENTRYPOINT
HEALTHCHECK
```

A useful trainer example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

Explain the build lifecycle:

```text
Source Code
     |
     v
Dockerfile
     |
     | docker build
     v
Build Context
     |
     v
Image Layers
     |
     v
Docker Image
```

## 2.2 CMD vs ENTRYPOINT

Show:

```dockerfile
CMD ["python", "app.py"]
```

versus:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Explain that `ENTRYPOINT` establishes the executable while `CMD` can supply default arguments.

## 2.3 ARG vs ENV

Build-time:

```dockerfile
ARG APP_VERSION
```

Runtime/environment:

```dockerfile
ENV APP_ENV=production
```

Demonstrate:

```bash
docker build \
  --build-arg APP_VERSION=2.0 \
  -t myapp:2.0 .
```

## 2.4 Image layers and caching

Run:

```bash
docker history myapp:1.0
```

Explain why dependency files are often copied before application source.

Bad:

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

Better:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```

Small source changes can then avoid invalidating the dependency-install layer.

---

# LAB 1 — Containerize a real application

Build a simple Python Flask application.

### app.py

```python
from flask import Flask
import socket

app = Flask(__name__)

@app.route("/")
def home():
    return {
        "message": "Docker DevOps Training",
        "hostname": socket.gethostname()
    }

@app.route("/health")
def health():
    return {"status": "healthy"}

app.run(host="0.0.0.0", port=5000)
```

### requirements.txt

```text
flask
```

### Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

Build:

```bash
docker build -t devops-flask:v1 .
```

Run:

```bash
docker run -d \
  --name flask-app \
  -p 5000:5000 \
  devops-flask:v1
```

Test:

```bash
curl localhost:5000
curl localhost:5000/health
```

Students should now understand:

```text
Python Code
   ↓
Dockerfile
   ↓
docker build
   ↓
devops-flask:v1
   ↓
docker run
   ↓
Container
   ↓
localhost:5000
```

---

# Module 3 — Docker Volumes

Start with:

> Containers should normally be considered replaceable. Application data often cannot be.

Explain Docker storage types:

```text
Docker Storage
     |
     +------ Container writable layer
     |
     +------ Bind Mount
     |
     +------ Named Volume
     |
     +------ tmpfs
```

## Named volume

```bash
docker volume create app-data
docker volume ls
docker volume inspect app-data
```

Attach:

```bash
docker run -d \
  --name nginx-volume \
  -p 8081:80 \
  -v app-data:/usr/share/nginx/html \
  nginx
```

## Bind mount

```bash
mkdir website

echo "<h1>Docker Bind Mount Demo</h1>" > website/index.html
```

Run:

```bash
docker run -d \
  --name bind-demo \
  -p 8082:80 \
  -v "$(pwd)/website:/usr/share/nginx/html:ro" \
  nginx
```

Explain the difference:

| Bind Mount                  | Named Volume                         |
| --------------------------- | ------------------------------------ |
| Host path controlled        | Docker-managed                       |
| Good for development/config | Good for persistent application data |
| Host filesystem visible     | More portable Docker abstraction     |

---

# LAB 2 — Persistent database

Use PostgreSQL.

Create:

```bash
docker volume create postgres-data
```

Run:

```bash
docker run -d \
  --name postgres-db \
  -e POSTGRES_USER=devops \
  -e POSTGRES_PASSWORD=Training123 \
  -e POSTGRES_DB=training \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:16
```

Check:

```bash
docker logs postgres-db
```

Enter:

```bash
docker exec -it postgres-db \
  psql -U devops -d training
```

Create:

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

INSERT INTO students(name)
VALUES ('Docker Student');

SELECT * FROM students;
```

Exit and delete the container:

```bash
docker rm -f postgres-db
```

Ask:

> Is our database gone?

Recreate using the **same volume** and verify the record remains.

This creates a memorable demonstration of persistence.

---

# Module 4 — Docker Networking

Explain:

```text
Container A
     |
     |
Docker Network
     |
     |
Container B
```

Cover:

* bridge
* host
* none
* user-defined bridge
* container DNS
* port publishing

Inspect defaults:

```bash
docker network ls
```

Create:

```bash
docker network create app-network
```

Inspect:

```bash
docker network inspect app-network
```

## Container DNS

Run:

```bash
docker run -d \
  --name nginx1 \
  --network app-network \
  nginx
```

Run another diagnostic container:

```bash
docker run --rm \
  --network app-network \
  busybox \
  ping -c 3 nginx1
```

Explain that Docker DNS resolves:

```text
nginx1
   ↓
Container IP
```

Students should use names rather than hard-coded container IP addresses.

---

# LAB 3 — Application + Database Network

Architecture:

```text
                    Browser
                       |
                    :5000
                       |
                       v
                +-------------+
                | Flask App   |
                +-------------+
                       |
                    db:5432
                       |
                Docker Network
                       |
                       v
                +-------------+
                | PostgreSQL  |
                +-------------+
                       |
                       v
                  DB Volume
```

Create:

```bash
docker network create production-net
```

Run PostgreSQL:

```bash
docker run -d \
  --name db \
  --network production-net \
  -e POSTGRES_PASSWORD=Training123 \
  postgres:16
```

Explain the key production concept:

The application should connect to:

```text
db:5432
```

not to a manually discovered container IP.

Trainer question:

> What happens to a container IP when the container is recreated?

This leads naturally into service discovery and eventually Kubernetes Services.

---

# Module 5 — Docker Compose

Tell students:

> Running five containers using five huge `docker run` commands becomes difficult to maintain.

Docker Compose lets us define the application stack declaratively.

Architecture:

```text
docker-compose.yml
        |
        +---- Web
        |
        +---- API
        |
        +---- Database
        |
        +---- Network
        |
        +---- Volumes
```

Introduce:

```yaml
services:
volumes:
networks:
```

Useful commands:

```bash
docker compose up
docker compose up -d
docker compose ps
docker compose logs
docker compose logs -f
docker compose stop
docker compose down
```

---

# LAB 4 — Production-style multi-container stack

Use Flask + PostgreSQL.

### compose.yaml

```yaml
services:

  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_NAME: training
      DB_USER: devops
      DB_PASSWORD: Training123
    depends_on:
      - db
    networks:
      - app-network

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: training
      POSTGRES_USER: devops
      POSTGRES_PASSWORD: Training123
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
```

Start:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs -f
```

Stop:

```bash
docker compose down
```

Then:

```bash
docker compose up -d
```

Verify database persistence.

### Trainer discussion

Now ask:

> How would you manage 100 containers across 10 servers?

Answer:

“This is where an orchestrator such as Kubernetes becomes necessary.”

---

# Module 6 — Azure Container Registry

Now transition from local Docker to cloud DevOps.

Explain:

```text
Developer
   |
docker build
   |
   v
Local Image
   |
docker push
   |
   v
Azure Container Registry
   |
   +----------------+
   |                |
   v                v
 AKS          Other runtimes
```

Use the official [Azure Container Registry documentation](https://learn.microsoft.com/azure/container-registry/?utm_source=chatgpt.com) as supporting reference.

---

# LAB 5 — Push application to ACR

Login:

```bash
az login
```

Variables:

```bash
RG=docker-training-rg
ACR=YOUR_UNIQUE_ACR_NAME
LOCATION=centralindia
```

Create resource group:

```bash
az group create \
  --name $RG \
  --location $LOCATION
```

Create ACR:

```bash
az acr create \
  --resource-group $RG \
  --name $ACR \
  --sku Basic
```

Login:

```bash
az acr login --name $ACR
```

Find login server:

```bash
az acr show \
  --name $ACR \
  --query loginServer \
  --output tsv
```

Tag:

```bash
docker tag \
  devops-flask:v1 \
  $ACR.azurecr.io/devops-flask:v1
```

Push:

```bash
docker push \
  $ACR.azurecr.io/devops-flask:v1
```

List repositories:

```bash
az acr repository list \
  --name $ACR \
  --output table
```

Show students Azure Portal → ACR → Repositories.

Connect this directly to:

```text
CI Pipeline
     ↓
docker build
     ↓
docker push
     ↓
ACR
```

---

# Module 7 — Docker Security

This section should focus on practices students can immediately apply.

## Don't run as root unnecessarily

Instead of relying on root, create/use a non-root user where the application permits it.

Example:

```dockerfile
RUN useradd -m appuser

USER appuser
```

## Don't store secrets in Dockerfiles

Bad:

```dockerfile
ENV DB_PASSWORD=MyPassword
```

Better conceptually:

```text
Secret Manager
     ↓
Runtime configuration
     ↓
Container
```

In Azure environments, connect this to Key Vault and workload identity patterns rather than baking credentials into images.

## Use minimal images

For example:

```dockerfile
FROM python:3.12-slim
```

instead of unnecessarily large general-purpose images.

## Pin versions

Avoid relying blindly on:

```dockerfile
FROM python:latest
```

Prefer a controlled version such as:

```dockerfile
FROM python:3.12-slim
```

For stronger reproducibility, production teams can pin image digests.

## Vulnerability scanning

Discuss tools such as:

* Trivy
* Microsoft Defender for Cloud/container registry scanning capabilities
* Snyk
* Docker Scout

Example development workflow with Trivy:

```bash
trivy image devops-flask:v1
```

## Other security topics

Cover:

* least privilege
* read-only filesystems where practical
* resource limits
* trusted base images
* image scanning
* patching/rebuilding images
* secret management
* SBOMs
* image signing/provenance
* avoiding unnecessary packages

### Security exercise

Give students this Dockerfile:

```dockerfile
FROM ubuntu:latest

ENV DB_PASSWORD=SuperSecret123

RUN apt-get update
RUN apt-get install -y python3

COPY . /app

CMD ["python3", "/app/app.py"]
```

Ask:

> Find at least five problems.

Then collectively improve it.

---

# Module 8 — Docker to AKS

This is the final production-style module.

Explain the transition:

```text
Laptop
  |
  | docker build
  v
Docker Image
  |
  | docker push
  v
ACR
  |
  | image pull
  v
AKS
  |
  +-----------------------+
  | Pod                   |
  |   Container           |
  +-----------------------+
             |
          Service
             |
             v
          User
```

Use [AKS documentation](https://learn.microsoft.com/azure/aks/?utm_source=chatgpt.com) as a reference for the current Azure workflow.

---

# LAB 6 — Deploy Docker application to AKS

Create AKS:

```bash
AKS=docker-training-aks
```

```bash
az aks create \
  --resource-group $RG \
  --name $AKS \
  --node-count 2 \
  --generate-ssh-keys \
  --attach-acr $ACR
```

Get credentials:

```bash
az aks get-credentials \
  --resource-group $RG \
  --name $AKS
```

Verify:

```bash
kubectl get nodes
```

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-flask
spec:
  replicas: 2

  selector:
    matchLabels:
      app: devops-flask

  template:
    metadata:
      labels:
        app: devops-flask

    spec:
      containers:
        - name: devops-flask
          image: YOUR_ACR.azurecr.io/devops-flask:v1

          ports:
            - containerPort: 5000

          readinessProbe:
            httpGet:
              path: /health
              port: 5000

          livenessProbe:
            httpGet:
              path: /health
              port: 5000
```

### service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: devops-flask-service

spec:

  type: LoadBalancer

  selector:
    app: devops-flask

  ports:
    - port: 80
      targetPort: 5000
```

Deploy:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Verify:

```bash
kubectl get pods
```

Then:

```bash
kubectl get deployments
```

And:

```bash
kubectl get services
```

Watch for the external IP:

```bash
kubectl get svc devops-flask-service -w
```

Once assigned:

```text
Browser
   ↓
AKS LoadBalancer
   ↓
Service
   ↓
Pod 1 ── Flask Container
Pod 2 ── Flask Container
```

Now make the connection:

> The application we created during the Docker lab is now running as containers orchestrated by Kubernetes in Azure.

---

# Final production challenge — V1 → V2 deployment

Finish with a realistic release.

Developer modifies:

```python
return {
    "message": "Docker Training Version 2"
}
```

Build:

```bash
docker build -t devops-flask:v2 .
```

Tag:

```bash
docker tag \
  devops-flask:v2 \
  $ACR.azurecr.io/devops-flask:v2
```

Push:

```bash
docker push \
  $ACR.azurecr.io/devops-flask:v2
```

Update AKS:

```bash
kubectl set image \
  deployment/devops-flask \
  devops-flask=$ACR.azurecr.io/devops-flask:v2
```

Watch rollout:

```bash
kubectl rollout status deployment/devops-flask
```

History:

```bash
kubectl rollout history deployment/devops-flask
```

Then teach rollback:

```bash
kubectl rollout undo deployment/devops-flask
```

This is an excellent end-of-day demonstration because students see the complete software-delivery lifecycle.

---

# Complete lab journey

The six labs progressively build on each other:

| Lab       | Production skill                                     |
| --------- | ---------------------------------------------------- |
| **Lab 1** | Containerize Flask application                       |
| **Lab 2** | Persist PostgreSQL data using volumes                |
| **Lab 3** | Connect application/database using Docker networking |
| **Lab 4** | Run multi-container stack using Compose              |
| **Lab 5** | Tag and push application image to ACR                |
| **Lab 6** | Deploy ACR image to AKS + perform V2 rollout         |

So the student effectively builds one evolving project:

```text
                    FULL-DAY PROJECT

Python Flask Application
          |
          v
      Dockerfile
          |
          v
      Docker Image
          |
     +----+-----+
     |          |
     v          v
  Network     Volume
     |          |
     +----+-----+
          |
          v
 Flask + PostgreSQL
          |
          v
   Docker Compose
          |
          v
       ACR
          |
          v
        AKS
          |
          v
   Deployment
          |
          v
      Service
          |
          v
   Application
          |
          v
      V1 → V2
          |
          v
   Rollout / Rollback
```

## Trainer troubleshooting drill

During each lab, deliberately introduce one problem rather than having everything succeed immediately.

For example, use a wrong port and ask students to troubleshoot with:

```bash
docker ps
docker logs <container>
docker inspect <container>
```

Use an incorrect image name:

```text
ImagePullBackOff
```

during the Kubernetes portion and troubleshoot with:

```bash
kubectl describe pod <pod>
```

Use:

```bash
kubectl logs <pod>
```

Then deliberately break V2 and demonstrate:

```bash
kubectl rollout undo deployment/devops-flask
```

This turns the workshop from a command-following exercise into a DevOps troubleshooting session.

## End-of-day knowledge check

Students should be able to answer these without notes:

1. Image vs container?
2. Dockerfile vs Compose file?
3. `CMD` vs `ENTRYPOINT`?
4. `COPY` vs `ADD`?
5. `ARG` vs `ENV`?
6. Why do Dockerfile layers matter?
7. What does `-p 8080:80` mean?
8. Why use volumes?
9. Bind mount vs named volume?
10. Why use user-defined networks?
11. How does container DNS work?
12. Why use Docker Compose?
13. Why do we need ACR?
14. Why shouldn't secrets be stored in images?
15. Why avoid running containers as root?
16. Why scan images?
17. Docker Compose vs Kubernetes?
18. How does AKS pull an image from ACR?
19. What is an AKS Deployment?
20. How do you roll back a failed deployment?

### Final message to trainees

> “Docker isn't just about learning `docker run`. In DevOps, the important skill is understanding the complete software supply chain.”

```text
CODE
 ↓
DOCKERFILE
 ↓
BUILD
 ↓
TEST
 ↓
SCAN
 ↓
IMAGE
 ↓
ACR
 ↓
AKS
 ↓
MONITOR
 ↓
UPDATE
 ↓
ROLLBACK
```

That should be the main architecture students remember from the full-day workshop.
