# Day 26: Docker Containers Interview Questions and Answers

## Purpose

This guide contains the six interview questions covered in the Day 26 Containers session. Each section includes:

- A clear interview-ready answer
- A practical command example
- Important points to mention
- A short answer for rapid revision

---

## 1. Describe the five-stage containerization assembly line and the purpose of each stage

A practical five-stage containerization lifecycle is:

```text
Code and Dockerfile -> Build -> Image -> Run as Container -> Publish to Registry
```

### Stage 1: Code and Dockerfile

The application source code and supporting files are prepared. A `Dockerfile` defines the base image, dependencies, configuration, files to copy, exposed ports, and startup behavior.

Example:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

### Stage 2: Build

Docker reads the Dockerfile and produces an image:

```bash
docker build -t day26-webserver:v1 .
```

The build should be repeatable so the same source and Dockerfile can produce a consistent environment.

### Stage 3: Image

The image is a reusable, versioned template. It contains the application, runtime, required files, and filesystem layers. An image is not the running application.

Inspect local images:

```bash
docker images
```

### Stage 4: Run as a container

A container is a runtime instance of an image:

```bash
docker run -d --name day26-container -p 81:80 day26-webserver:v1
```

Multiple containers can be created from the same image. Each container receives its own writable layer.

### Stage 5: Publish to a registry

The image is tagged and pushed to a registry such as Docker Hub so other machines and deployment systems can pull it:

```bash
docker tag day26-webserver:v1 YOUR_USERNAME/day26-webserver:v1
docker push YOUR_USERNAME/day26-webserver:v1
```

### Interview-ready answer

> The five stages are code and Dockerfile, build, image, container execution, and registry publication. The Dockerfile defines the environment, the build produces an image, the image is the reusable template, `docker run` creates a container instance, and a registry distributes versioned images to other systems.

### Rapid answer

> Dockerfile, build, image, run, and registry. The image is the reusable template, while the container is its runtime instance.

---

## 2. Compare foreground and detached container execution. When would you use each?

### Foreground execution

A container runs in the foreground when its main process remains attached to the terminal:

```bash
docker run --name nginx-foreground -p 81:80 nginx:alpine
```

Characteristics:

- Application output appears in the current terminal.
- The terminal remains attached to the container's main process.
- `Ctrl+C` normally sends an interrupt to the attached process.
- Useful for learning, short tests, direct observation, and debugging startup failures.

### Detached execution

The `-d` option runs the container in the background:

```bash
docker run -d --name nginx-detached -p 81:80 nginx:alpine
```

Characteristics:

- Docker returns the container ID.
- The terminal is immediately available for other commands.
- The container continues running in the background.
- Useful for web servers, APIs, databases, and other long-running services.

Check a detached container:

```bash
docker ps
```

Read its logs:

```bash
docker logs nginx-detached
```

Follow logs continuously:

```bash
docker logs -f nginx-detached
```

Stop it:

```bash
docker stop nginx-detached
```

### Interview-ready answer

> Foreground execution attaches the container's main process to the current terminal, making it useful for direct observation and debugging. Detached execution uses `-d` and runs the container in the background, which is appropriate for long-running services. A detached container can be monitored using `docker ps` and `docker logs`.

### Rapid answer

> Foreground is attached to the terminal and is useful for debugging. Detached mode uses `-d`, runs in the background, and is preferred for services.

---

## 3. Explain host-to-container port mapping using `-p 81:80`

The syntax is:

```text
-p HOST_PORT:CONTAINER_PORT
```

Therefore:

```bash
docker run -d -p 81:80 nginx:alpine
```

means:

```text
Host port 81 -> Container port 80
```

The traffic path is:

```text
Browser or client
       |
       | http://localhost:81
       v
Docker host port 81
       |
       | Port forwarding
       v
Container port 80
       |
       v
Nginx web server
```

Nginx listens on port 80 inside the container. The user accesses it through port 81 on the host:

```text
http://localhost:81
```

Verify the mapping:

```bash
docker ps
```

The output should show a mapping similar to:

```text
0.0.0.0:81->80/tcp
```

Test it:

```bash
curl http://localhost:81
```

### Important distinction

`EXPOSE 80` in a Dockerfile documents the expected container port. It does not automatically publish that port on the host. The `-p` option publishes and maps it.

### Interview-ready answer

> In `-p 81:80`, port 81 is the port exposed on the Docker host and port 80 is the port used by the application inside the container. A request to `localhost:81` is forwarded by Docker to port 80 in the container.

### Rapid answer

> `-p 81:80` means host port 81 forwards traffic to container port 80.

---

## 4. What does `docker exec -it` do, and how is the host different from the container environment?

Example:

```bash
docker exec -it day26-container sh
```

Command breakdown:

- `docker exec` runs a new command inside an already running container.
- `-i` keeps standard input open.
- `-t` allocates a pseudo-terminal.
- `day26-container` identifies the target container.
- `sh` starts a shell inside the container.

After entering the container, commands execute in the container environment:

```sh
cat /usr/share/nginx/html/index.html
```

Exit the shell with:

```sh
exit
```

### Host environment

The host is the machine or virtual machine running Docker. It contains:

- The operating system and host filesystem
- Docker Engine
- Local images, containers, volumes, and networks
- Host processes and network interfaces

### Container environment

The container is an isolated runtime environment created by Docker. It has:

- Its own filesystem view
- Its own process namespace
- Its own network namespace unless configured otherwise
- A writable container layer over the image layers
- Resource and security boundaries configured by Docker and the host kernel

Containers share the host kernel, so a container is not the same as a full virtual machine.

### Important difference from `docker run`

- `docker run` creates and starts a new container from an image.
- `docker exec` runs an additional command inside an existing running container.

### Interview-ready answer

> `docker exec -it` starts an interactive command inside an existing running container. The `-i` option keeps input open and `-t` provides a terminal. The host runs Docker and owns the underlying operating system, while the container has an isolated filesystem, processes, networking context, and writable layer, although it shares the host kernel.

### Rapid answer

> `docker exec -it` opens an interactive command or shell inside a running container. The host runs Docker, while the container provides an isolated application environment.

---

## 5. Why is `docker commit` needed after modifying a running container?

An image is used to create a container:

```text
day26-webserver:v1 -> day26-container
```

If the running container is modified using:

```bash
docker exec -it day26-container sh
```

those changes are written to the container's writable layer. They do not modify the original `day26-webserver:v1` image.

To capture the modified state as another reusable image, run:

```bash
docker commit day26-container day26-webserver:v2
```

Verify the images:

```bash
docker images day26-webserver
```

The expected result includes both versions:

```text
day26-webserver   v2
day26-webserver   v1
```

A fresh container can then be created from the committed image:

```bash
docker run -d --name day26-v2 -p 81:80 day26-webserver:v2
```

### What `docker commit` does not do

- It does not change the original image.
- It does not update the project's Dockerfile.
- It does not automatically push the new image to a registry.
- It does not provide the same reproducibility as rebuilding from source and a Dockerfile.

### Production best practice

`docker commit` is useful for this lab, investigation snapshots, and capturing an experimental state. For normal production delivery, edit the source or Dockerfile and rebuild:

```bash
docker build -t day26-webserver:v2 .
```

This makes the change reviewable and repeatable.

### Interview-ready answer

> Changes made inside a running container are stored in that container's writable layer and do not alter the original image. `docker commit` captures the modified container state as a new image. For production, updating the Dockerfile or source and rebuilding is normally preferred because it is repeatable and auditable.

### Rapid answer

> `docker commit` captures a modified container's writable state as a new image without changing the original image.

---

## 6. Explain the image tag format and the steps to publish an image to Docker Hub

A Docker Hub image reference normally follows this format:

```text
DOCKERHUB_USERNAME/REPOSITORY:TAG
```

Example:

```text
wilson123/day26-webserver:v2
```

Components:

- `wilson123` is the Docker Hub namespace or username.
- `day26-webserver` is the repository name.
- `v2` is the tag identifying the version.

If no tag is supplied, Docker generally uses `latest`. `latest` is only a tag name. It does not guarantee that an image is the newest or most appropriate version.

### Step 1: Build or identify the local image

```bash
docker images
```

For this lab, the source image is:

```text
day26-webserver:v2
```

### Step 2: Authenticate

```bash
docker login
```

Use the correct Docker Hub username and password or personal access token.

### Step 3: Tag the local image

```bash
docker tag day26-webserver:v2 YOUR_USERNAME/day26-webserver:v2
```

### Step 4: Verify the tag

```bash
docker images
```

The image should appear with the registry-ready name:

```text
YOUR_USERNAME/day26-webserver   v2
```

### Step 5: Push the image

```bash
docker push YOUR_USERNAME/day26-webserver:v2
```

A successful push displays layer status and an image digest.

### Step 6: Verify in Docker Hub

1. Sign in to Docker Hub with the same account.
2. Open **My Hub** or **Repositories**.
3. Open the `day26-webserver` repository.
4. Open **Tags**.
5. Confirm that `v2` is listed.

### Step 7: Test the published image

Pull it:

```bash
docker pull YOUR_USERNAME/day26-webserver:v2
```

Run it:

```bash
docker run -d --name published-day26 -p 82:80 YOUR_USERNAME/day26-webserver:v2
```

Test it:

```bash
curl http://localhost:82
```

### Common push error

If the push returns `requested access to the resource is denied`, verify:

- The Docker login account is correct.
- The image name begins with the authenticated username or permitted organization.
- The repository name and tag are spelled correctly.
- The account has push permission for the namespace.

### Interview-ready answer

> A Docker Hub image name uses `username/repository:tag`, such as `wilson123/day26-webserver:v2`. I authenticate with `docker login`, add the Docker Hub namespace using `docker tag`, push with `docker push`, and verify the tag in the repository's Tags section. Other systems can then retrieve the exact version using `docker pull`.

### Rapid answer

> Use `username/repository:tag`, then run `docker login`, `docker tag`, and `docker push`. Verify the version under the repository's Tags section.

---

# Complete command sequence

```bash
# Build version 1
docker build -t day26-webserver:v1 .

# Run version 1
docker run -d --name day26-container -p 81:80 day26-webserver:v1

# Enter the running container
docker exec -it day26-container sh

# After modifying the container, create version 2
docker commit day26-container day26-webserver:v2

# Tag version 2 for Docker Hub
docker tag day26-webserver:v2 YOUR_USERNAME/day26-webserver:v2

# Authenticate and push
docker login
docker push YOUR_USERNAME/day26-webserver:v2

# Verify that the published image can be pulled
docker pull YOUR_USERNAME/day26-webserver:v2
```

---

# One-minute revision table

| Question | Key response |
|---|---|
| Five-stage lifecycle | Dockerfile, build, image, run, registry |
| Foreground execution | Attached to the terminal and useful for debugging |
| Detached execution | Uses `-d` and runs in the background |
| `-p 81:80` | Host port 81 forwards to container port 80 |
| `docker exec -it` | Runs an interactive command inside an existing container |
| Host vs container | Host runs Docker; container isolates the application environment |
| `docker commit` | Captures the modified container state as a new image |
| Image reference | `username/repository:tag` |
| Publish workflow | `docker login`, `docker tag`, `docker push` |
| Docker Hub verification | Repository -> Tags -> confirm `v2` |

---

# Practical follow-up questions

## What is the difference between an image and a container?

An image is a read-only template. A container is a runtime instance of that image with a writable layer.

## Does `EXPOSE 80` publish port 80 on the host?

No. `EXPOSE` documents the intended container port. Use `-p`, such as `-p 81:80`, to publish it on the host.

## Can multiple containers use the same image?

Yes. Docker can create multiple independent containers from the same image, provided host resources and port assignments do not conflict.

## Can two containers both map to host port 81?

Not on the same host IP at the same time. The second container normally fails because the port is already allocated. Use a different host port, such as `-p 82:80`.

## Does stopping a container delete it?

No. `docker stop` stops the process. `docker rm` removes the stopped container.

## Does removing a container remove its image?

No. Containers and images are separate Docker objects. Use `docker rmi IMAGE` to remove an image when no dependent container blocks removal.

## What is the difference between `docker exec` and `docker attach`?

`docker exec` starts a new command inside the running container. `docker attach` connects the terminal to the container's existing main process.

## Why are version tags important?

Version tags such as `v1` and `v2` allow teams and deployment systems to reference known image versions rather than relying only on the ambiguous `latest` tag.

---

# Final interview summary

The complete lifecycle is:

```text
Create Dockerfile
      |
      v
Build image v1
      |
      v
Run container with 81:80 mapping
      |
      v
Modify container with docker exec
      |
      v
Commit modified state as image v2
      |
      v
Tag as username/repository:v2
      |
      v
Push and verify in Docker Hub
```

A strong interview answer should explain both the command and the concept behind it. Mention isolation, image immutability, the container writable layer, explicit port mapping, versioned tags, and the preference for reproducible Dockerfile builds in production.
