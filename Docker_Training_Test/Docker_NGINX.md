For training, it is useful to demonstrate NGINX in **two ways**: first run the official NGINX image directly, then create **your own NGINX image using a Dockerfile**. The second method clearly teaches the Image → Container lifecycle.

## Hands-on Lab: Create an NGINX Docker Image and Run It

### Architecture

```text
index.html + Dockerfile
        │
        ▼
   docker build
        │
        ▼
 custom-nginx:v1
   Docker Image
        │
        │ docker run
        ▼
   NGINX Container
        │
     Port 80
        │
        │ -p 8080:80
        ▼
localhost:8080
        │
        ▼
     Browser
```

### Step 1 — Verify Docker

Open **Ubuntu/WSL terminal** with Docker Desktop running.

```bash
docker --version
docker info
```

A quick verification:

```bash
docker run hello-world
```

If this succeeds, Docker Engine is accessible.

### Step 2 — Create the project directory

```bash
mkdir nginx-docker-demo
cd nginx-docker-demo
```

Check:

```bash
pwd
```

Your project will eventually contain:

```text
nginx-docker-demo/
├── Dockerfile
└── index.html
```

### Step 3 — Create a custom web page

Create:

```bash
nano index.html
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker DevOps Training</title>
</head>

<body>
    <h1>Welcome to Docker Training</h1>
    <h2>NGINX is running inside a Docker Container</h2>
    <p>Image: custom-nginx:v1</p>
    <p>DevOps Hands-on Lab</p>
</body>
</html>
```

Save with `Ctrl+O`, Enter, then exit with `Ctrl+X`.

Verify:

```bash
cat index.html
```

### Step 4 — Create the Dockerfile

```bash
nano Dockerfile
```

Add:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Save and exit.

Check:

```bash
cat Dockerfile
```

### Step 5 — Explain the Dockerfile

The first instruction:

```dockerfile
FROM nginx:alpine
```

means:

> Use the NGINX Alpine image as our base image.

Then:

```dockerfile
COPY index.html /usr/share/nginx/html/index.html
```

copies our custom page into the NGINX document root.

Finally:

```dockerfile
EXPOSE 80
```

documents that the application listens on TCP port 80.

Important training point: `EXPOSE 80` **does not publish port 80 to the host**. We will do that with `docker run -p`.

### Step 6 — Check the files

```bash
ls -l
```

Expected:

```text
Dockerfile
index.html
```

Now we're ready to build.

### Step 7 — Build the Docker image

Run:

```bash
docker build -t custom-nginx:v1 .
```

Break the command down for students:

```text
docker build -t custom-nginx:v1 .
       │      │        │       │
       │      │        │       └── Build context
       │      │        └────────── Image tag
       │      └─────────────────── Image name
       └────────────────────────── Build image
```

The final `.` is important. It tells Docker to use the current directory as the build context.

Representative output:

```text
[+] Building ...
 => [internal] load build definition from Dockerfile
 => [internal] load metadata for nginx:alpine
 => [1/2] FROM nginx:alpine
 => [2/2] COPY index.html /usr/share/nginx/html/index.html
 => exporting to image
 => naming to docker.io/library/custom-nginx:v1
```

### Step 8 — Verify the image

```bash
docker images
```

You should see something similar to:

```text
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
custom-nginx    v1        abc123def456   20 seconds ago   ...
nginx           alpine    ...            ...              ...
```

You have now completed:

```text
Dockerfile
    ↓
docker build
    ↓
custom-nginx:v1
```

At this point, **you have an image, not a running application**.

### Step 9 — Inspect the image

```bash
docker image inspect custom-nginx:v1
```

You can also show its layer history:

```bash
docker history custom-nginx:v1
```

This is useful for explaining:

```text
nginx:alpine base image
        ↓
COPY index.html
        ↓
custom-nginx:v1
```

### Step 10 — Create and run the container

Now run:

```bash
docker run -d \
  --name nginx-web \
  -p 8080:80 \
  custom-nginx:v1
```

Explain each parameter:

| Parameter          | Meaning                    |
| ------------------ | -------------------------- |
| `docker run`       | Create and start container |
| `-d`               | Detached/background mode   |
| `--name nginx-web` | Container name             |
| `-p 8080:80`       | Host 8080 → container 80   |
| `custom-nginx:v1`  | Image                      |

The most important part is:

```text
-p 8080:80
    │    │
    │    └── Container port
    │
    └─────── Host port
```

So:

```text
Browser
   │
   ▼
localhost:8080
   │
   ▼
Docker Host :8080
   │
   ▼
Container :80
   │
   ▼
NGINX
```

### Step 11 — Verify the container

```bash
docker ps
```

Representative output:

```text
CONTAINER ID   IMAGE             STATUS       PORTS                  NAMES
abc123def456   custom-nginx:v1   Up 10 sec    0.0.0.0:8080->80/tcp   nginx-web
```

This confirms the container is running.

### Step 12 — Test from the command line

```bash
curl http://localhost:8080
```

You should receive your custom HTML.

### Step 13 — Test from browser

Open:

```text
http://localhost:8080
```

You should see:

```text
Welcome to Docker Training

NGINX is running inside a Docker Container

Image: custom-nginx:v1

DevOps Hands-on Lab
```

### Step 14 — Check NGINX logs

Run:

```bash
docker logs nginx-web
```

Refresh the browser several times and run it again:

```bash
docker logs nginx-web
```

Students should see NGINX access requests similar to:

```text
172.x.x.x - - "GET / HTTP/1.1" 200 ...
172.x.x.x - - "GET / HTTP/1.1" 200 ...
```

For live logs:

```bash
docker logs -f nginx-web
```

Refresh the browser and observe requests appear.

Press `Ctrl+C` to stop following the logs; this does not stop the container.

### Step 15 — Enter the running container

```bash
docker exec -it nginx-web sh
```

Because we're using Alpine, `sh` is more appropriate than assuming Bash exists.

Inside the container:

```bash
hostname
```

Then:

```bash
ls /usr/share/nginx/html
```

View the page:

```bash
cat /usr/share/nginx/html/index.html
```

You should see the HTML that was copied during the image build.

Exit:

```bash
exit
```

This gives students a clear picture:

```text
Host

nginx-docker-demo/
├── Dockerfile
└── index.html
        │
        │ docker build
        ▼
Docker Image
custom-nginx:v1
        │
        │ docker run
        ▼
Container
nginx-web
        │
        └── /usr/share/nginx/html/index.html
```

### Step 16 — Inspect the container

```bash
docker inspect nginx-web
```

During training, don't explain the entire JSON. Highlight:

```text
Image
State
NetworkSettings
IPAddress
Ports
Mounts
```

For example, retrieve only the container IP:

```bash
docker inspect \
  -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  nginx-web
```

### Step 17 — Stop the container

```bash
docker stop nginx-web
```

Check:

```bash
docker ps
```

It disappears from the running-container list.

But:

```bash
docker ps -a
```

shows the stopped container.

This demonstrates an important distinction:

```text
docker stop
     ↓
Container stopped
     ↓
Container still exists
```

### Step 18 — Start it again

```bash
docker start nginx-web
```

Check:

```bash
docker ps
```

Then refresh:

```text
http://localhost:8080
```

The application works again.

### Step 19 — Remove the container

```bash
docker rm -f nginx-web
```

Check:

```bash
docker ps -a
```

The container is gone.

But check:

```bash
docker images
```

You should still have:

```text
custom-nginx    v1
```

This is another critical concept:

```text
IMAGE
custom-nginx:v1
      │
      ├── Container A
      ├── Container B
      └── Container C
```

Removing a container does **not** automatically remove its image.

### Step 20 — Run multiple containers from one image

This is a good final classroom demonstration.

```bash
docker run -d \
  --name nginx1 \
  -p 8081:80 \
  custom-nginx:v1
```

Then:

```bash
docker run -d \
  --name nginx2 \
  -p 8082:80 \
  custom-nginx:v1
```

Check:

```bash
docker ps
```

You now have:

```text
                 custom-nginx:v1
                    IMAGE
                   /     \
                  /       \
                 ▼         ▼
             nginx1      nginx2
             :80         :80
               │           │
               ▼           ▼
Host :8081              Host :8082
```

Open:

```text
http://localhost:8081
http://localhost:8082
```

Both containers serve the same application because both were created from the same image.

## Troubleshooting Exercise

If the application does not open, teach students to avoid immediately deleting and recreating everything.

Use this sequence:

```text
Application not working
        ↓
docker ps -a
        ↓
Container running?
        ↓
docker logs <container>
        ↓
Check port mapping
        ↓
docker inspect <container>
        ↓
docker exec
        ↓
Check NGINX internally
```

Commands:

```bash
docker ps -a
docker logs nginx-web
docker inspect nginx-web
docker exec -it nginx-web sh
```

Inside:

```bash
ps
ls /usr/share/nginx/html
cat /usr/share/nginx/html/index.html
```

A common error is:

```bash
docker run -d \
  --name nginx-web \
  -p 8080:8080 \
  custom-nginx:v1
```

This is wrong for the default NGINX configuration because NGINX listens on **80 inside the container**.

Correct:

```bash
-p 8080:80
```

## Cleanup

Remove containers:

```bash
docker rm -f nginx1 nginx2
```

Remove custom image:

```bash
docker rmi custom-nginx:v1
```

Check:

```bash
docker images
docker ps -a
```

The complete lab can be summarized as:

```text
Dockerfile + index.html
          ↓
     docker build
          ↓
   custom-nginx:v1
      Docker Image
          ↓
      docker run
          ↓
      nginx-web
       Container
          ↓
    NGINX Port 80
          ↓
    -p 8080:80
          ↓
   localhost:8080
          ↓
       Browser
```

For a training session, this lab is especially useful because it demonstrates **Dockerfile → Base Image → Build → Custom Image → Container → Port Mapping → Logs → Exec → Stop/Start → Image vs Container → Multiple Containers from One Image** in one short exercise.
