Below is a **trainer-ready speaking script** aligned with the 20-slide Docker PPT. You can use it almost word-for-word while delivering the session.

## Slide 1 — Docker for DevOps

**Trainer Script**

“Good morning everyone. Today we are going to learn Docker from a DevOps perspective.

Our objective is not only to understand Docker theoretically. We are going to see how Docker works, execute Docker commands using Ubuntu/WSL, use Docker Desktop, build our own Docker image, and finally run our application inside a container.

By the end of this session, you should understand the complete flow:

Application code → Dockerfile → Docker image → Docker container → Registry → Kubernetes.”

**Ask the class:** “Has anyone faced a situation where an application works on a developer's laptop but fails on another server?”

Use their answers to introduce the problem Docker solves.

---

## Slide 2 — Learning Objectives

**Trainer Script**

“Before starting, these are our learning objectives.

First, we will understand what containers are and why Docker became important in DevOps.

Then we'll understand the difference between Docker images and containers.

We'll work with the Docker CLI and learn commands such as `docker pull`, `docker run`, `docker ps`, `docker logs` and `docker exec`.

Most importantly, we'll create a Dockerfile and build our own Docker image.

Finally, we'll connect Docker with a real DevOps CI/CD and Kubernetes workflow.”

**Trainer tip:** Tell students not to memorize commands initially. Focus on understanding the lifecycle.

---

## Slide 3 — Why Docker?

**Trainer Script**

“Before Docker, applications were commonly installed directly on servers or virtual machines.

Imagine our developer builds an application using a particular runtime and set of libraries.

The application works perfectly on the developer's machine.

When we deploy it to QA, it fails because QA has another version of a dependency.

Then somebody says the famous sentence:

‘But it works on my machine.’

Docker helps address this deployment consistency problem.

Instead of giving operations only application code, we package the application, runtime and required dependencies into an image.

We can then use that same image across environments.”

Draw this on the board:

```text
Developer
    ↓
Application
+ Runtime
+ Dependencies
    ↓
Docker Image
    ↓
DEV → TEST → UAT → PROD
```

**Key statement:** “Build once, deploy the same artifact consistently.”

---

## Slide 4 — Containers vs Virtual Machines

**Trainer Script**

“Containers and virtual machines provide isolation differently.

With a virtual machine, we have physical infrastructure, a hypervisor, and multiple guest operating systems.

Each VM therefore carries an operating system.

Containers work differently. Containers share the host kernel while isolating application processes and filesystems.

That's one reason containers can generally start much faster and have less overhead than full virtual machines.”

Explain:

```text
Virtual Machines

Hardware
   ↓
Host / Hypervisor
   ↓
---------------------
VM1       VM2
OS        OS
App       App
```

Versus:

```text
Containers

Infrastructure
     ↓
Host OS
     ↓
Container Runtime
     ↓
-----------------------
Container  Container
   App        App
```

**Ask:** “Does Docker completely replace virtual machines?”

Expected answer: **No.**

Explain that Docker containers frequently run **inside VMs** in cloud environments.

---

## Slide 5 — Core Docker Concepts

**Trainer Script**

“There are five terms I want everyone to remember.”

### Dockerfile

“A Dockerfile contains instructions for building our application image.”

### Image

“An image is the packaged, immutable template used to create containers.”

### Container

“A container is a running instance of an image.”

Use an analogy:

```text
Dockerfile = Recipe
Image      = Prepared package/template
Container  = Running instance
```

### Registry

“A registry stores and distributes Docker images.”

Examples include Docker Hub and Azure Container Registry.

### Docker Engine

“The Docker Engine is responsible for building images and managing containers.”

---

## Slide 6 — Docker Architecture

**Trainer Script**

“Now let's understand what happens behind a Docker command.”

Write:

```text
Docker CLI
    ↓
Docker Engine
    ↓
Images
    ↓
Containers
```

“And externally we can have a registry.”

```text
             Registry
                ↕
Docker CLI → Docker Engine
                ↓
              Images
                ↓
            Containers
```

“If I execute:

`docker run nginx`

Docker first checks whether the required image is available locally.

If it isn't available, Docker downloads it from the configured registry.

Docker then creates and starts a container from that image.”

---

## Slide 7 — Docker Desktop + Ubuntu/WSL

**Trainer Script**

“For today's lab, I am using Windows with Docker Desktop and Ubuntu through WSL.

Docker Desktop gives us an easy graphical interface.

Ubuntu gives us a Linux command-line environment.

During this training I recommend learning Docker through the command line rather than depending entirely on Docker Desktop.”

Now switch from PPT to Ubuntu.

Say:

“From this point onward, we'll frequently switch between theory and live demonstrations.”

---

# Slide 8 — DEMO 1: Verify Docker

Tell students:

“Before working with Docker, always confirm that the environment is working.”

Run:

```bash
docker --version
```

Explain the output.

Then:

```bash
docker version
```

Say:

“Notice that Docker can report client and server/engine information.”

Run:

```bash
docker info
```

Then:

```bash
docker run hello-world
```

Before pressing Enter, ask:

“What do you think will happen?”

Then explain the process:

```text
docker run hello-world
        ↓
Check local image
        ↓
Image unavailable
        ↓
Pull image
        ↓
Create container
        ↓
Start container
        ↓
Print output
```

Run:

```bash
docker images
```

Then:

```bash
docker ps -a
```

Point out the exited `hello-world` container.

---

# Slide 9 — Image Lifecycle

**Trainer Script**

“Containers come from images, so let's first understand image management.”

Run:

```bash
docker pull nginx
```

Say:

“We have explicitly asked Docker to download the NGINX image.”

Then:

```bash
docker images
```

Explain columns such as repository, tag, image ID and size.

Run:

```bash
docker image inspect nginx
```

Explain that `inspect` provides detailed metadata.

Mention:

```bash
docker rmi nginx
```

“But don't remove it yet because we'll use NGINX in the next demonstration.”

---

# Slide 10 — Container Lifecycle

**Trainer Script**

“An image itself isn't the running application. We create a container from it.”

Run:

```bash
docker run -d --name web nginx
```

Explain:

```text
docker run
   ↓
Create + Start

-d
   ↓
Detached/background

--name web
   ↓
Container name

nginx
   ↓
Image
```

Run:

```bash
docker ps
```

Then:

```bash
docker stop web
```

Run:

```bash
docker ps
```

Ask:

“Has the container been deleted?”

Then:

```bash
docker ps -a
```

Explain:

“Stopped does not mean deleted.”

Start it:

```bash
docker start web
```

Finally:

```bash
docker rm -f web
```

---

# Slide 11 — DEMO 2: Run NGINX

Tell students:

“Now we're going to run something that we can actually access from a browser.”

Run:

```bash
docker run -d \
--name webserver \
-p 8080:80 \
nginx
```

Explain each parameter carefully.

Then:

```bash
docker ps
```

Open browser:

```text
localhost:8080
```

Say:

“This NGINX website isn't running directly on my Windows machine. NGINX is running inside our container.”

Now generate some browser traffic and run:

```bash
docker logs webserver
```

Students should see HTTP requests.

Then:

```bash
docker exec -it webserver bash
```

If Bash isn't available for a chosen minimal image, use:

```bash
docker exec -it webserver sh
```

Inside the container:

```bash
hostname
```

```bash
ls
```

```bash
cd /usr/share/nginx/html
```

```bash
cat index.html
```

Exit:

```bash
exit
```

---

# Slide 12 — Port Mapping

**Trainer Script**

“One of the most important concepts here is port mapping.”

Draw:

```text
Laptop
localhost:8080
      |
      | -p 8080:80
      ↓
Docker Container
NGINX :80
```

Explain:

“The syntax is:

`HOST_PORT:CONTAINER_PORT`.”

Run another container:

```bash
docker run -d --name nginx2 -p 8081:80 nginx
```

Now demonstrate:

```text
localhost:8080
localhost:8081
```

Explain:

“Both containers use port 80 internally, but we publish them through different host ports.”

---

# Slide 13 — Dockerfile Fundamentals

Return to PPT.

**Trainer Script**

“Until now we have used images created by someone else.

But DevOps engineers need to know how to package their own applications.

That's where Dockerfiles come in.”

Show:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Explain line-by-line.

### FROM

“Select our base image.”

### COPY

“Copy our application into the image.”

### EXPOSE

“Documents the intended application port.”

Important point:

“`EXPOSE 80` does not by itself make the application available through port 8080 on our laptop. Actual publishing happens using `docker run -p`.”

---

# Slide 14 — HANDS-ON LAB

Tell students:

“Now everyone should perform this section themselves.”

Create project:

```bash
mkdir docker-demo
cd docker-demo
```

Create application:

```bash
nano index.html
```

Use:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker DevOps Training</title>
</head>

<body>

<h1>Hello from Docker!</h1>

<h2>DevOps Docker Training</h2>

<p>This application is running inside a Docker container.</p>

</body>
</html>
```

Create:

```bash
nano Dockerfile
```

Add:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Before proceeding, verify:

```bash
ls
```

Expected:

```text
Dockerfile
index.html
```

---

# Slide 15 — Build Docker Image

**Trainer Script**

“Now we have application code and a Dockerfile.

The next step is creating an image.”

Run:

```bash
docker build -t devops-docker-demo:v1 .
```

Explain:

```text
docker build
      ↓
Build image

-t
      ↓
Tag/name image

devops-docker-demo
      ↓
Repository/image name

:v1
      ↓
Tag

.
      ↓
Current directory/build context
```

After completion:

```bash
docker images
```

Point out:

```text
devops-docker-demo    v1
```

Say:

“We have now converted our application into a Docker image.”

---

# Slide 16 — Run and Validate

Run:

```bash
docker run -d \
--name devops-web \
-p 8082:80 \
devops-docker-demo:v1
```

Then:

```bash
docker ps
```

Open:

```text
localhost:8082
```

Tell students:

“If you can see your custom HTML page, you've successfully containerized your first application.”

Check:

```bash
docker logs devops-web
```

Enter:

```bash
docker exec -it devops-web sh
```

Inside:

```bash
cat /usr/share/nginx/html/index.html
```

Exit:

```bash
exit
```

### Mini challenge

Modify the HTML:

```html
<h1>Docker Application Version 2</h1>
```

Build:

```bash
docker build -t devops-docker-demo:v2 .
```

Show:

```bash
docker images
```

Explain why image tags matter in CI/CD.

---

# Slide 17 — Docker Volumes

**Trainer Script**

“Containers should generally be treated as disposable.

But what if the application needs persistent data?”

Draw:

```text
Container
    |
    | writes
    ↓
Volume
    |
Persistent Data
```

Run:

```bash
docker volume create demo-data
```

Then:

```bash
docker volume ls
```

And:

```bash
docker volume inspect demo-data
```

Explain:

“The container lifecycle and data lifecycle don't necessarily have to be the same.

This becomes especially important for databases and stateful applications.”

Connect this concept to Kubernetes Persistent Volumes.

---

# Slide 18 — Docker Networking

**Trainer Script**

“Containers also need to communicate with other containers.”

Create:

```bash
docker network create devops-network
```

Check:

```bash
docker network ls
```

Run:

```bash
docker run -d \
--name web1 \
--network devops-network \
nginx
```

Then:

```bash
docker run -d \
--name web2 \
--network devops-network \
nginx
```

Inspect:

```bash
docker network inspect devops-network
```

Explain:

“On a user-defined Docker network, Docker provides DNS-based name resolution between containers.”

Draw:

```text
        devops-network
              |
      -----------------
      |               |
    web1             web2
      |               |
    nginx            nginx
```

---

# Slide 19 — Docker in CI/CD

This is one of the most important slides for a DevOps audience.

**Trainer Script**

“Everything we've manually done today is what CI/CD pipelines automate.”

Explain:

```text
Developer
    |
    v
GitHub / Azure Repos
    |
    v
CI Pipeline
    |
    +--- Build
    |
    +--- Test
    |
    +--- docker build
    |
    v
Docker Image
    |
    v
Container Registry
    |
    v
Kubernetes / AKS
```

Give an Azure example:

```text
Developer
    ↓
Azure Repos / GitHub
    ↓
Azure DevOps / GitHub Actions
    ↓
docker build
    ↓
Azure Container Registry
    ↓
AKS
```

Explain:

“The CI pipeline should build a versioned image.”

For example:

```bash
docker build -t myapp:1.0 .
```

Then an appropriately authenticated pipeline pushes the image to its registry.

“The deployment system then pulls that specific image version.”

Connect today's lab:

```text
index.html
    ↓
Dockerfile
    ↓
docker build
    ↓
devops-docker-demo:v1
    ↓
Registry
    ↓
AKS
```

Say:

“So Docker isn't an isolated skill. It sits directly in the middle of modern DevOps delivery.”

---

# Slide 20 — Troubleshooting, Recap & Challenge

**Trainer Script**

“Before finishing, these are the first commands I normally use when troubleshooting Docker.”

### Container isn't running

```bash
docker ps -a
```

### Application failed

```bash
docker logs <container>
```

### Need configuration details

```bash
docker inspect <container>
```

### Need resource usage

```bash
docker stats
```

### Need to enter the container

```bash
docker exec -it <container> sh
```

Give students this troubleshooting sequence:

```text
Application unavailable
        ↓
docker ps
        ↓
Container running?
        ↓
docker logs
        ↓
Check port mapping
        ↓
docker inspect
        ↓
docker exec
        ↓
Check application inside container
```

Finish by asking students these five questions:

1. What is the difference between an image and container?
2. What does `docker run` do?
3. What does `-p 8080:80` mean?
4. Why do we need a Dockerfile?
5. Why do we need a container registry?

Then finish with:

“Today we manually completed this workflow:

```text
Code
 ↓
Dockerfile
 ↓
docker build
 ↓
Image
 ↓
docker run
 ↓
Container
 ↓
Application
```

In a production DevOps environment, we extend it to:

```text
Developer
    ↓
Git
    ↓
CI/CD
    ↓
Docker Build
    ↓
Image
    ↓
ACR / Registry
    ↓
AKS / Kubernetes
    ↓
Production
```

That is the main Docker workflow I want you to remember.”

### Suggested trainer timing

| Section                               |         Time |
| ------------------------------------- | -----------: |
| Slides 1–3: Introduction              |       10 min |
| Slides 4–6: Architecture & concepts   |       15 min |
| Slides 7–10: CLI demonstration        |       20 min |
| Slides 11–12: NGINX live demo         |       15 min |
| Slides 13–16: Dockerfile hands-on lab |   **30 min** |
| Slides 17–18: Volumes & networking    |       15 min |
| Slide 19: CI/CD + AKS                 |       10 min |
| Slide 20: Troubleshooting + Q&A       |       15 min |
| **Total**                             | **~2 hours** |

For a **full-day Docker training**, I would expand this next into a trainer package with **Module 2: Dockerfile deep dive, Module 3: Docker volumes, Module 4: Docker networking, Module 5: Docker Compose, Module 6: ACR, Module 7: Docker security, and Module 8: Docker-to-AKS deployment**, with 5–6 production-style hands-on labs.
