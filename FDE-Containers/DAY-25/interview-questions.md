# Day 25 - Containers & Dockerfile Interview Questions

**Module:** FDE-Containers  
**Day:** 25  
**Focus:** Docker, Dockerfile, AWS EC2, container lifecycle, networking, permissions, and CI/CD

---

## 1. Explain the evolution from manual EC2 configuration to containerized releases.

### Short interview answer

With manual EC2 deployment, an engineer provisions a server, connects over SSH, installs packages, copies application files, configures services, and starts the application. This works, but repeated manual configuration can cause environment drift.

With containers, the operating requirements are described in a `Dockerfile`. Docker builds those instructions into a versioned image, and containers are created from that same image across environments.

```text
Manual EC2 deployment
        |
        v
SSH into server
        |
        v
Install dependencies
        |
        v
Copy/configure application
        |
        v
Start service manually

        becomes

Dockerfile
    |
    v
docker build
    |
    v
Versioned Docker image
    |
    v
docker run
    |
    v
Repeatable container
```

The main shift is from **configuring servers manually** to **building versioned deployment artifacts**.

### Day-25 example

Instead of repeatedly installing Apache directly on every EC2 host, the lab encoded the application runtime in a Dockerfile:

```dockerfile
FROM ubuntu:latest

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y apache2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY index.html /var/www/html/index.html

EXPOSE 80

CMD ["apache2ctl", "-D", "FOREGROUND"]
```

Then the image was built as:

```bash
sudo docker build -t custom-apache:v1 .
```

### Key phrase to remember

> Manual deployment configures servers. Containerized deployment builds immutable, versioned artifacts that can be deployed consistently.

---

## 2. Explain the local, host, and guest boundaries when Docker runs on a remote AWS EC2 instance.

### Short interview answer

There are three important boundaries:

- **Local machine:** My laptop, browser, SSH client, and local development tools.
- **Host:** The remote Ubuntu EC2 instance running Docker Engine.
- **Guest/container:** The isolated Docker container running the application, in this case Apache.

The Day-25 request path was:

```text
Local browser
     |
     | HTTP
     v
EC2 public address :81
     |
     | AWS networking / Security Group
     v
Ubuntu EC2 Docker host :81
     |
     | Docker -p 81:80
     v
Docker container :80
     |
     v
Apache
```

AWS networking controls whether traffic can reach the EC2 host. Docker networking controls how traffic is forwarded from a host port to a container port.

### Key phrase to remember

> Local is my computer, host is the EC2 machine running Docker, and guest is the container running Apache.

---

## 3. A user gets permission denied connecting to `/var/run/docker.sock`. What diagnosis and remediation should be used?

### Short interview answer

I first determine whether Docker itself is unhealthy or whether only the current user lacks permission.

Compare:

```bash
docker ps
```

with:

```bash
sudo docker ps
```

If `docker ps` fails with permission denied but `sudo docker ps` works, the Docker daemon is probably healthy and the issue is likely access to the Docker Unix socket.

Check the socket:

```bash
ls -l /var/run/docker.sock
```

Check user/group membership:

```bash
groups
```

or:

```bash
id
```

When appropriate, add the user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Then start a new login session and verify:

```bash
docker ps
```

### Security consideration

Membership in the `docker` group provides powerful access to the Docker host, so it should only be granted to trusted users when appropriate.

### Key phrase to remember

> If sudo works and the normal command does not, check socket ownership and user group membership before blaming the Docker daemon.

---

## 4. What is the difference between a Docker image and a running container, and why does a container need a foreground process?

### Docker image

A Docker image is a reusable, read-only template containing the filesystem and instructions needed to create a container.

Example:

```text
custom-apache:v1
```

### Docker container

A container is a runtime instance created from an image.

```text
Dockerfile
    |
    | docker build
    v
Docker image
custom-apache:v1
    |
    | docker run
    v
Container
apache-container
```

Multiple containers can be created from one image.

### Why does the foreground process matter?

A container's lifecycle is tied to its main process. When that process exits, the container exits.

Apache often runs as a background daemon on a traditional server. Inside this container, Apache was intentionally kept in the foreground:

```dockerfile
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

Conceptually:

```text
Foreground process alive
        |
        v
Container alive

Foreground process exits
        |
        v
Container exits
```

### Key phrase to remember

> An image is a template; a container is a running instance. The container stays alive while its main process stays alive.

---

## 5. Explain the Apache Dockerfile layers and the meaning of `-p 81:80`.

The Day-25 Dockerfile contains the following logical steps.

### Base image

```dockerfile
FROM ubuntu:latest
```

Provides the Ubuntu base filesystem.

### Install Apache

```dockerfile
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y apache2
```

Installs the Apache HTTP server and required packages.

### Copy application content

```dockerfile
COPY index.html /var/www/html/index.html
```

Copies the custom web page into Apache's web root.

### Document the application port

```dockerfile
EXPOSE 80
```

Documents that the application expects to listen on container port 80.

`EXPOSE 80` does not by itself publish the port through the Docker host.

### Define the startup process

```dockerfile
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

Starts Apache as the container's foreground process.

### What does `-p 81:80` mean?

Docker uses:

```text
-p HOST_PORT:CONTAINER_PORT
```

Therefore:

```bash
-p 81:80
```

means:

```text
EC2 host :81
     |
     | Docker forwarding
     v
Container :80
     |
     v
Apache
```

The Day-25 lab verified the mapping with:

```bash
sudo docker port apache-container
```

which showed the container's TCP port 80 published on host port 81.

### Key phrase to remember

> `81:80` means host port 81 forwards to container port 80. Apache still listens on port 80 inside the container.

---

## 6. Why must teardown follow stop, remove container, then remove image? How does this connect to CI/CD?

### Teardown sequence

The resources have a dependency relationship:

```text
Image
  |
  v
Container created from image
  |
  v
Running process
```

Cleanup reverses that lifecycle.

### Step 1: Stop the running container

```bash
sudo docker stop apache-container
```

This stops the container's main process.

### Step 2: Remove the container

```bash
sudo docker rm apache-container
```

This removes the container object that references the image.

### Step 3: Remove the image

```bash
sudo docker rmi custom-apache:v1
```

Now the custom image can be removed after its dependent container has been removed.

```text
Running container
       |
       | docker stop
       v
Stopped container
       |
       | docker rm
       v
Container removed
       |
       | docker rmi
       v
Image removed
```

### Connection to CI/CD

Container-based CI/CD uses versioned images as release artifacts.

A common lifecycle is:

```text
Git commit
    |
    v
Build image
    |
    v
Run tests
    |
    v
Tag image
    |
    v
Push to registry
    |
    v
Deploy new containers
    |
    v
Health verification
    |
    v
Retire previous containers
    |
    v
Prune unused images later
```

Instead of manually modifying a production container, a pipeline normally builds a new image version and deploys new containers from that artifact.

For example:

```text
Application v1
     |
     v
custom-apache:v1
     |
     v
Container v1

New code
     |
     v
custom-apache:v2
     |
     v
Container v2
     |
     v
Verify v2
     |
     v
Retire v1
```

This improves repeatability and supports rollback to a previously known image version.

### Key phrase to remember

> Stop the running process, remove the container that depends on the image, then remove the unused image. CI/CD extends the same lifecycle using versioned immutable images.

---

# Rapid Revision Table

| Interview topic | Remember this |
|---|---|
| Manual EC2 vs containers | Configure servers → build versioned artifacts |
| Local / host / guest | Laptop → EC2 → container |
| `docker.sock` permission | Compare normal vs `sudo`, then inspect socket/groups |
| Image vs container | Template → running instance |
| Foreground process | Main process exits → container exits |
| `EXPOSE 80` | Describes intended container port |
| `-p 81:80` | Host 81 → container 80 |
| Docker teardown | Stop → remove container → remove image |
| CI/CD | Build → test → tag → push → deploy → verify → retire |

---

# Commands to Remember

```bash
# Docker status
docker --version
sudo docker ps
sudo docker ps -a

# Permission diagnosis
docker ps
sudo docker ps
ls -l /var/run/docker.sock
groups
id

# Build
sudo docker build -t custom-apache:v1 .

# Run
sudo docker run -d --name apache-container -p 81:80 custom-apache:v1

# Inspect / troubleshoot
sudo docker port apache-container
sudo docker logs apache-container
sudo docker inspect apache-container

# Cleanup
sudo docker stop apache-container
sudo docker rm apache-container
sudo docker rmi custom-apache:v1
```

---

# Final Interview Strategy

For each answer, use this pattern:

1. **Define the concept.**
2. **Explain why it matters.**
3. **Use the Day-25 Apache-on-EC2 lab as evidence.**
4. **Connect the concept to production or CI/CD.**

Example:

> `-p 81:80` follows Docker's `HOST_PORT:CONTAINER_PORT` format. In my Day-25 lab, the EC2 Docker host accepted traffic on port 81 and Docker forwarded it to Apache on port 80 inside the container. I verified the mapping with `docker port`. In production, the same port-publishing concept is combined with cloud networking, load balancers, security controls, and automated deployment.

---

# Short Revision Cheat Sheet

Use this section for a 2-minute review before an interview.

## Core Concepts

| Topic | Quick answer |
|---|---|
| **Docker image** | Read-only, versioned template used to create containers. |
| **Docker container** | Running instance of an image. |
| **Dockerfile** | Instructions used by `docker build` to create an image. |
| **Foreground process** | Main container process. If it exits, the container exits. |
| **`EXPOSE 80`** | Documents the intended container port; it does not publish it externally. |
| **`-p 81:80`** | Host port `81` forwards to container port `80`. |
| **Docker host** | The EC2/Ubuntu machine running Docker Engine. |
| **Guest/container** | Isolated runtime where Apache runs. |
| **`docker.sock`** | Unix socket commonly used by the Docker CLI to communicate with the Docker daemon. |
| **Cleanup order** | Stop container → remove container → remove image. |

## Day-25 Request Flow

```text
Laptop / Browser
      |
      v
AWS EC2 :81
      |
      | Docker -p 81:80
      v
Container :80
      |
      v
Apache
      |
      v
index.html
```

Remember:

```text
AWS Security Group = controls access TO the EC2 host
Docker -p           = controls host-to-container port forwarding
Apache              = listens on port 80 inside the container
```

## Dockerfile in 5 Lines

```text
FROM    → choose base image
RUN     → install/configure dependencies
COPY    → add application files
EXPOSE  → document container port
CMD     → define the default foreground process
```

## Commands to Memorize

```bash
# Check
docker --version
sudo docker ps
sudo docker ps -a

# Build
sudo docker build -t custom-apache:v1 .

# Run
sudo docker run -d --name apache-container -p 81:80 custom-apache:v1

# Verify
sudo docker port apache-container
sudo docker logs apache-container
sudo docker inspect apache-container

# Cleanup
sudo docker stop apache-container
sudo docker rm apache-container
sudo docker rmi custom-apache:v1
```

## `docker.sock` Troubleshooting

```text
docker ps fails
     |
     v
Try sudo docker ps
     |
     +-- sudo also fails → check Docker service/daemon
     |
     +-- sudo works → likely user/socket permission issue
                         |
                         v
              ls -l /var/run/docker.sock
              groups
              id
```

Possible remediation when appropriate:

```bash
sudo usermod -aG docker $USER
```

Then start a new login session and verify again.

## Six Interview Answers in One Line

1. **Manual EC2 → Containers:** Configure servers manually → build repeatable, versioned images.
2. **Local / Host / Guest:** Laptop → EC2 Docker host → application container.
3. **Docker permission denied:** Compare normal vs `sudo`, then inspect socket ownership and groups.
4. **Image vs Container:** Image = template; container = running instance; PID 1/main process keeps it alive.
5. **`-p 81:80`:** EC2 host port 81 → container port 80 → Apache.
6. **Cleanup + CI/CD:** Stop → remove container → remove image; pipelines build and deploy new image versions instead of manually changing production.

## CI/CD Mental Model

```text
Git Commit
    ↓
Build Image
    ↓
Test
    ↓
Tag
    ↓
Push to Registry
    ↓
Deploy
    ↓
Health Check
    ↓
Retire Old Version
```

## Interview Formula

For almost every Docker question:

```text
DEFINE IT
   ↓
EXPLAIN WHY IT MATTERS
   ↓
GIVE THE DAY-25 APACHE EXAMPLE
   ↓
CONNECT IT TO PRODUCTION / CI/CD
```

### Example

> `-p 81:80` uses Docker's host-port-to-container-port format. In my Day-25 lab, EC2 accepted traffic on port 81 and Docker forwarded it to Apache on port 80 inside the container. I verified the mapping with `docker port`. In production, the same concept works alongside cloud networking and automated deployment.

## Final Memory Map

```text
Dockerfile
    ↓
IMAGE
    ↓
CONTAINER
    ↓
PORT MAPPING
    ↓
VERIFY
    ↓
DEBUG
    ↓
STOP
    ↓
REMOVE CONTAINER
    ↓
REMOVE IMAGE
    ↓
AUTOMATE WITH CI/CD
```
