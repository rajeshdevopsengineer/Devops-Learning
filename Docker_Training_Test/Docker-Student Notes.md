# Docker for DevOps — Student Study Notes

These notes are designed to accompany the **full-day Docker training** and the six hands-on labs. They focus on concepts, commands, troubleshooting, and interview revision.

## Module 1 — Docker Fundamentals

### What is Docker?

Docker is a container platform used to **build, package, distribute, and run applications in containers**.

A common deployment problem is:

```text
Developer Laptop
     ↓
Application works
     ↓
Move to Test/Production
     ↓
Different runtime/dependencies
     ↓
Application fails
```

Docker packages the application with its required runtime and dependencies into an **image**.

```text
Application
+ Runtime
+ Libraries
+ Dependencies
       ↓
 Docker Image
       ↓
DEV → TEST → UAT → PROD
```

### Important Docker terminology

| Term           | Meaning                                  |
| -------------- | ---------------------------------------- |
| Dockerfile     | Instructions for building an image       |
| Image          | Packaged, read-only application template |
| Container      | Running instance of an image             |
| Docker Engine  | Builds/runs/manages containers           |
| Registry       | Stores and distributes images            |
| Volume         | Persistent container data storage        |
| Network        | Communication between containers         |
| Docker Compose | Defines multi-container applications     |

### Containers vs Virtual Machines

```text
VIRTUAL MACHINES                 CONTAINERS

Hardware                         Infrastructure
   ↓                                  ↓
Hypervisor                        Host OS
   ↓                                  ↓
VM1       VM2                    Container Runtime
 ↓         ↓                           ↓
Guest OS  Guest OS              ┌──────┴──────┐
 ↓         ↓                    Container    Container
App       App                     App          App
```

VMs provide a complete guest OS. Containers use OS-level isolation and share the relevant host kernel.

Containers therefore tend to be:

* faster to start
* smaller than full VMs
* easier to package and distribute
* well suited to CI/CD and microservices

Containers **do not eliminate VMs**. For example, AKS worker nodes commonly run as Azure VMs.

### Docker architecture

```text
                 Registry
                  ↑    ↓
                  │
Docker CLI → Docker Engine
                  │
           ┌──────┴───────┐
           ↓              ↓
         Images       Containers
```

When you execute:

```bash
docker run nginx
```

Docker roughly performs:

```text
Check local image
      ↓
Image available?
   ↙       ↘
 Yes       No
  │         ↓
  │      Registry
  │         ↓
  └────── Pull
          ↓
    Create container
          ↓
     Start process
```

### Essential commands

```bash
docker --version
docker version
docker info
```

Images:

```bash
docker images
docker pull nginx
docker image inspect nginx
docker rmi nginx
```

Containers:

```bash
docker ps
docker ps -a

docker run nginx
docker stop <container>
docker start <container>
docker restart <container>
docker rm <container>
```

Logs:

```bash
docker logs <container>
docker logs -f <container>
```

Enter a running container:

```bash
docker exec -it <container> bash
```

or for minimal images:

```bash
docker exec -it <container> sh
```

---

# Module 2 — Dockerfile Deep Dive

A Dockerfile contains the instructions used to build an image.

```text
Source Code
    +
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Container
```

### Important Dockerfile instructions

| Instruction   | Purpose                           |
| ------------- | --------------------------------- |
| `FROM`        | Base image                        |
| `WORKDIR`     | Working directory                 |
| `COPY`        | Copy files into image             |
| `RUN`         | Execute command while building    |
| `ARG`         | Build-time variable               |
| `ENV`         | Environment variable              |
| `EXPOSE`      | Documents application port        |
| `USER`        | Runtime user                      |
| `CMD`         | Default runtime command/arguments |
| `ENTRYPOINT`  | Defines executable/entry point    |
| `HEALTHCHECK` | Defines container health test     |

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

### FROM

```dockerfile
FROM python:3.12-slim
```

Defines the base image.

Avoid relying blindly on:

```dockerfile
FROM python:latest
```

Production environments should use controlled versions, and stricter environments may pin digests.

### WORKDIR

```dockerfile
WORKDIR /app
```

Sets the working directory.

### COPY

```dockerfile
COPY app.py .
```

Copies files from the build context into the image.

### RUN

```dockerfile
RUN pip install -r requirements.txt
```

Runs during **image build**.

### CMD

```dockerfile
CMD ["python", "app.py"]
```

Provides the default command when the container starts.

Remember:

```text
RUN → build time
CMD → runtime default
```

### ARG vs ENV

```dockerfile
ARG APP_VERSION=1.0
ENV APP_ENV=production
```

`ARG` is primarily used during image build.

`ENV` establishes an environment variable in the image/container environment.

### EXPOSE vs `-p`

```dockerfile
EXPOSE 5000
```

does **not** publish the application to your laptop.

Actual port publishing happens with:

```bash
docker run -p 5000:5000 myapp
```

### Image layers and caching

Docker builds images as layers.

```text
FROM python
     ↓
WORKDIR /app
     ↓
COPY requirements.txt
     ↓
RUN pip install
     ↓
COPY application
     ↓
Final Image
```

A useful pattern is:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```

If only application code changes, Docker may reuse the dependency layer.

Inspect layers:

```bash
docker history myimage:v1
```

### `.dockerignore`

Similar conceptually to `.gitignore`.

Example:

```text
.git
__pycache__
*.log
.env
node_modules
```

This keeps unnecessary files out of the build context.

---

# Lab 1 — Containerize Flask

Application:

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

`requirements.txt`:

```text
flask
```

Build:

```bash
docker build -t devops-flask:v1 .
```

Verify:

```bash
docker images
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

Expected health response:

```text
{"status":"healthy"}
```

---

# Module 3 — Docker Volumes

Containers should generally be considered replaceable.

The problem:

```text
Container
   ↓
Application writes data
   ↓
Container deleted
   ↓
Container writable data lost
```

For persistent data:

```text
Container
    ↓
 Volume
    ↓
Persistent Data
```

### Storage options

Docker commonly uses:

* container writable layer
* bind mounts
* named volumes
* tmpfs

### Named volumes

Create:

```bash
docker volume create app-data
```

List:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect app-data
```

Use:

```bash
docker run \
  -v app-data:/data \
  myimage
```

Syntax:

```text
volume-name:container-path
```

### Bind mounts

A host directory is mapped directly into a container.

```bash
docker run \
  -v "$(pwd)/website:/usr/share/nginx/html:ro" \
  nginx
```

`ro` means read-only from the container's perspective.

### Volume vs bind mount

| Named Volume                          | Bind Mount                     |
| ------------------------------------- | ------------------------------ |
| Docker-managed                        | Host path                      |
| Good for persistent application data  | Useful for development/config  |
| Docker controls location              | User controls host location    |
| Less coupled to host directory layout | Direct host filesystem mapping |

Important:

> Persistent storage is not automatically a backup.

Production data still requires proper backup, retention, restore testing and disaster recovery.

---

# Lab 2 — PostgreSQL Persistence

Create volume:

```bash
docker volume create postgres-data
```

Run PostgreSQL:

```bash
docker run -d \
  --name postgres-db \
  -e POSTGRES_USER=devops \
  -e POSTGRES_PASSWORD=Training123 \
  -e POSTGRES_DB=training \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:16
```

Connect:

```bash
docker exec -it postgres-db \
  psql -U devops -d training
```

Create data:

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

INSERT INTO students(name)
VALUES ('Docker Student');

SELECT * FROM students;
```

Delete container:

```bash
docker rm -f postgres-db
```

Recreate it using the **same volume**.

The table remains because:

```text
Container lifecycle ≠ Volume lifecycle
```

---

# Module 4 — Docker Networking

Applications often require multiple containers.

Example:

```text
Browser
   ↓
Frontend
   ↓
Backend/API
   ↓
Database
```

Docker networking enables these containers to communicate.

### View networks

```bash
docker network ls
```

### Create network

```bash
docker network create app-network
```

Inspect:

```bash
docker network inspect app-network
```

### Run container on network

```bash
docker run -d \
  --name web \
  --network app-network \
  nginx
```

### Docker DNS

Containers on a suitable user-defined network can communicate using names.

```text
app-network

Flask
  │
  │ db:5432
  ↓
PostgreSQL
container name = db
```

Application configuration:

```text
DB_HOST=db
DB_PORT=5432
```

Avoid:

```text
DB_HOST=172.x.x.x
```

because container IP addresses can change.

### Important distinction

Container-to-container communication:

```text
web → db:5432
```

doesn't inherently require:

```bash
-p 5432:5432
```

Publishing is required when the service needs to be exposed through the Docker host.

---

# Lab 3 — Docker Networking

Create:

```bash
docker network create production-net
```

Run NGINX:

```bash
docker run -d \
  --name web1 \
  --network production-net \
  nginx
```

Test name resolution:

```bash
docker run --rm \
  --network production-net \
  busybox \
  ping -c 3 web1
```

This demonstrates:

```text
web1
 ↓
Docker DNS
 ↓
Container address
```

---

# Module 5 — Docker Compose

Consider:

```text
Frontend
Backend
PostgreSQL
Redis
NGINX
```

Maintaining separate `docker run` commands becomes difficult.

Docker Compose lets you describe the application declaratively.

### Compose architecture

```text
compose.yaml
     │
 ┌───┼─────────┐
 ↓   ↓         ↓
Web  DB      Network
     ↓
   Volume
```

Example:

```yaml
services:

  web:
    build: .
    ports:
      - "5000:5000"

  db:
    image: postgres:16
```

### Important commands

Start:

```bash
docker compose up
```

Background:

```bash
docker compose up -d
```

Status:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs
docker compose logs -f web
```

Validate configuration:

```bash
docker compose config
```

Stop/remove stack:

```bash
docker compose down
```

Be careful with:

```bash
docker compose down -v
```

because `-v` removes declared volumes.

### `depends_on`

Example:

```yaml
depends_on:
  - db
```

This can control startup ordering, but **does not by itself guarantee that PostgreSQL is ready to accept application connections**.

Production-style applications should use health checks and retry logic.

---

# Lab 4 — Flask + PostgreSQL Compose

```yaml
services:

  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
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

Run:

```bash
docker compose config
docker compose up -d
docker compose ps
```

Architecture:

```text
Browser
   ↓
localhost:5000
   ↓
Flask
   ↓
db:5432
   ↓
PostgreSQL
   ↓
postgres-data
```

---

# Module 6 — Azure Container Registry

Local Docker images exist only on the machine where they were built unless they are distributed.

A registry solves this.

```text
Developer
   ↓
docker build
   ↓
Docker Image
   ↓
docker push
   ↓
Azure Container Registry
   ↓
AKS
```

[Azure Container Registry documentation](https://learn.microsoft.com/azure/container-registry/?utm_source=chatgpt.com)

### ACR image naming

```text
<registry>.azurecr.io/<repository>:<tag>
```

Example:

```text
companyacr.azurecr.io/devops-flask:v1
```

### Basic workflow

Login:

```bash
az login
```

Create resource group:

```bash
az group create \
  --name docker-training-rg \
  --location centralindia
```

Create ACR:

```bash
az acr create \
  --resource-group docker-training-rg \
  --name <unique-acr-name> \
  --sku Basic
```

Login:

```bash
az acr login --name <acr-name>
```

Tag:

```bash
docker tag \
  devops-flask:v1 \
  <acr-name>.azurecr.io/devops-flask:v1
```

Push:

```bash
docker push \
  <acr-name>.azurecr.io/devops-flask:v1
```

Verify:

```bash
az acr repository list \
  --name <acr-name> \
  --output table
```

### Production image tagging

Avoid depending only on:

```text
latest
```

Prefer identifiable release tags such as:

```text
myapp:1.0.0
myapp:1.0.1
myapp:build-154
myapp:<commit-sha>
```

For strict immutability, images can also be referenced by digest.

---

# Module 7 — Docker Security

A container running successfully doesn't automatically mean it is production-ready.

### 1. Avoid secrets in images

Bad:

```dockerfile
ENV DB_PASSWORD=SuperSecret123
```

Secrets should be provided securely at runtime using an appropriate secrets-management solution.

In Azure/Kubernetes environments, this may involve:

```text
Azure Key Vault
      ↓
Secure identity/secret integration
      ↓
Application
```

### 2. Avoid root where practical

Example:

```dockerfile
RUN useradd -m appuser

USER appuser
```

Principle:

```text
Least Privilege
```

### 3. Use controlled/minimal images

Instead of an unnecessarily large general-purpose base image, use an appropriate runtime image such as:

```dockerfile
FROM python:3.12-slim
```

### 4. Scan images

Common tools include:

* Trivy
* Docker Scout
* Snyk
* Microsoft security capabilities

Example:

```bash
trivy image devops-flask:v1
```

### 5. Additional security practices

Use:

* trusted base images
* controlled versions/digests
* regular rebuilding/patching
* `.dockerignore`
* non-root users
* runtime secret management
* vulnerability scanning
* read-only filesystems where feasible
* CPU/memory limits
* SBOMs
* image signing/provenance
* CI/CD security gates

---

# Module 8 — Docker to AKS

[Azure Kubernetes Service documentation](https://learn.microsoft.com/azure/aks/?utm_source=chatgpt.com)

Final architecture:

```text
Developer
   ↓
Source Code
   ↓
Dockerfile
   ↓
docker build
   ↓
Docker Image
   ↓
ACR
   ↓
AKS
   ↓
Deployment
   ↓
Pods
   ↓
Containers
   ↓
Service
   ↓
Users
```

### Create AKS

```bash
az aks create \
  --resource-group docker-training-rg \
  --name docker-training-aks \
  --node-count 2 \
  --generate-ssh-keys \
  --attach-acr <acr-name>
```

Get credentials:

```bash
az aks get-credentials \
  --resource-group docker-training-rg \
  --name docker-training-aks
```

Check:

```bash
kubectl get nodes
```

### Deployment

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
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get deployments
kubectl get pods
```

### Service

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

Apply:

```bash
kubectl apply -f service.yaml
```

Check:

```bash
kubectl get svc
```

Architecture:

```text
Internet
   ↓
Azure Load Balancer
   ↓
Kubernetes Service
   ↓
 ┌─────────────┐
 ↓             ↓
Pod 1         Pod 2
 ↓             ↓
Flask         Flask
```

---

# Lab 6 — V1 → V2 Deployment

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

Update Kubernetes:

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

Rollback:

```bash
kubectl rollout undo deployment/devops-flask
```

---

# Docker Troubleshooting Cheat Sheet

When a container doesn't work, use a systematic approach.

```text
Application unavailable
        ↓
docker ps -a
        ↓
Container running?
        ↓
docker logs
        ↓
Port correct?
        ↓
docker inspect
        ↓
Network/volume correct?
        ↓
docker exec
        ↓
Check application internally
```

Important commands:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
docker stats
docker exec -it <container> sh
docker network inspect <network>
docker volume inspect <volume>
```

For Compose:

```bash
docker compose ps
docker compose config
docker compose logs
docker compose logs <service>
```

For AKS:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events
```

### Common problems

| Problem               | First checks                           |
| --------------------- | -------------------------------------- |
| Container exits       | `docker ps -a`, `docker logs`          |
| Port not accessible   | `docker ps`, port mapping              |
| Image not found       | Image/tag/registry                     |
| Container DNS failure | Network membership                     |
| Data disappeared      | Volume mapping                         |
| Compose failure       | `docker compose config`, logs          |
| ACR push denied       | Login/RBAC/image tag                   |
| ImagePullBackOff      | Image name/tag, ACR access, Pod events |
| CrashLoopBackOff      | Logs, command, config, probes          |
| AKS app unavailable   | Pods, Service, endpoints/probes        |

---

# Docker Command Revision Sheet

```bash
# Information
docker version
docker info

# Images
docker pull nginx
docker images
docker image inspect nginx
docker history nginx
docker rmi nginx

# Containers
docker run
docker ps
docker ps -a
docker stop
docker start
docker restart
docker rm

# Troubleshooting
docker logs
docker inspect
docker stats
docker exec

# Build
docker build -t app:v1 .

# Volumes
docker volume create data
docker volume ls
docker volume inspect data

# Networks
docker network create app-net
docker network ls
docker network inspect app-net

# Compose
docker compose config
docker compose up -d
docker compose ps
docker compose logs
docker compose down

# Registry
docker tag
docker push
docker pull

# Kubernetes
kubectl get nodes
kubectl get pods
kubectl get deployments
kubectl get svc
kubectl describe pod
kubectl logs
kubectl rollout status
kubectl rollout history
kubectl rollout undo
```

# Final Revision — Remember This Architecture

```text
                  DEVOPS CONTAINER PIPELINE

Developer
    ↓
Source Control
    ↓
CI Pipeline
    ↓
Build + Test
    ↓
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
Security Scan
    ↓
Container Registry / ACR
    ↓
Deployment / GitOps
    ↓
AKS
    ↓
Deployment
    ↓
Pods
    ↓
Service / Ingress
    ↓
Application
    ↓
Monitoring
    ↓
Update / Rollback
```

The key idea for students is not merely memorizing Docker commands. Understand the lifecycle:

**Code → Build → Image → Container → Storage → Networking → Compose → Registry → Security → Kubernetes → Production.**
