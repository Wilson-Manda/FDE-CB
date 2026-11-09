# Day 25 - Docker & Dockerfile Flashcards

**Module:** FDE-Containers  
**Day:** 25  
**Focus:** Docker images, containers, Dockerfile, AWS EC2, networking, permissions, cleanup, and CI/CD

---

## How to Use

Read the **Front**, answer aloud without looking, then check the **Back**. Mark cards you miss and repeat them until you can answer each in 20-30 seconds.

---

## Card 1

**Front:** What is a Docker image?

**Back:** A Docker image is a read-only, versioned template containing the filesystem, dependencies, application files, and startup instructions used to create containers.

**Memory hook:** `Image = template`

---

## Card 2

**Front:** What is a Docker container?

**Back:** A Docker container is a running instance of an image. Multiple independent containers can be created from the same image.

**Memory hook:** `Container = running instance`

---

## Card 3

**Front:** What is the relationship between a Dockerfile, image, and container?

**Back:**

```text
Dockerfile
    |
    | docker build
    v
Image
    |
    | docker run
    v
Container
```

---

## Card 4

**Front:** What is a Dockerfile?

**Back:** A Dockerfile is a text file containing instructions Docker uses to build an image.

Typical Day-25 sequence:

```text
FROM -> RUN -> COPY -> EXPOSE -> CMD
```

---

## Card 5

**Front:** What does `FROM ubuntu:latest` do?

**Back:** It selects the Ubuntu `latest` image as the base filesystem and starting point for the custom image.

---

## Card 6

**Front:** What does `RUN` do in a Dockerfile?

**Back:** `RUN` executes commands while the image is being built and creates filesystem changes in the resulting image, such as installing Apache.

Example:

```dockerfile
RUN apt-get update && apt-get install -y apache2
```

---

## Card 7

**Front:** What does `COPY` do in a Dockerfile?

**Back:** `COPY` copies files from the Docker build context into the image.

Day-25 example:

```dockerfile
COPY index.html /var/www/html/index.html
```

---

## Card 8

**Front:** What does `EXPOSE 80` mean?

**Back:** It documents that the application expects to listen on container port 80. It does **not** by itself publish that port through the Docker host or internet.

**Memory hook:** `EXPOSE = document, not publish`

---

## Card 9

**Front:** What does `CMD` do?

**Back:** `CMD` defines the default process or command that runs when a container starts from the image.

Day-25 example:

```dockerfile
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

---

## Card 10

**Front:** Why does Apache run with `FOREGROUND` inside the container?

**Back:** The container's lifecycle is tied to its main process. Running Apache in foreground mode keeps that main process alive. When the main process exits, the container exits.

```text
Main process alive -> Container alive
Main process exits -> Container exits
```

---

## Card 11

**Front:** What does `docker build -t custom-apache:v1 .` mean?

**Back:**

- `docker build` builds an image.
- `-t custom-apache:v1` assigns repository/name `custom-apache` and tag `v1`.
- `.` uses the current directory as the build context.

---

## Card 12

**Front:** What does `docker run -d --name apache-container -p 81:80 custom-apache:v1` do?

**Back:** It creates and starts a container named `apache-container` from `custom-apache:v1`, runs it in detached mode, and publishes EC2 host port 81 to container port 80.

---

## Card 13

**Front:** What does `-d` mean in `docker run`?

**Back:** Detached mode. Docker starts the container in the background instead of attaching the terminal to its foreground output.

---

## Card 14

**Front:** What does `-p 81:80` mean?

**Back:** Docker uses `HOST_PORT:CONTAINER_PORT`.

```text
EC2 host :81
     |
     v
Container :80
     |
     v
Apache
```

**Memory hook:** `Outside 81 -> Inside 80`

---

## Card 15

**Front:** What is the difference between `EXPOSE 80` and `-p 81:80`?

**Back:** `EXPOSE 80` documents the intended container port. `-p 81:80` actually publishes container port 80 through host port 81.

---

## Card 16

**Front:** What are the local, host, and guest/container boundaries in the Day-25 lab?

**Back:**

```text
Local = laptop/browser
Host = Ubuntu AWS EC2 instance running Docker Engine
Guest = Docker container running Apache
```

---

## Card 17

**Front:** What was the Day-25 request path?

**Back:**

```text
Laptop / Browser
      |
      v
EC2 public address :81
      |
      v
AWS Security Group
      |
      v
EC2 Docker host :81
      |
      | Docker -p 81:80
      v
Container :80
      |
      v
Apache
```

---

## Card 18

**Front:** What is the difference between an AWS Security Group and Docker port publishing?

**Back:** The AWS Security Group controls whether network traffic can reach the EC2 host. Docker `-p` controls how traffic reaching a host port is forwarded to a container port.

---

## Card 19

**Front:** How do you verify that a container is running?

**Back:**

```bash
sudo docker ps
```

Use this to view currently running containers, their status, and published ports.

---

## Card 20

**Front:** How do you verify the published port for `apache-container`?

**Back:**

```bash
sudo docker port apache-container
```

Day-25 expected mapping:

```text
80/tcp -> 0.0.0.0:81
80/tcp -> [::]:81
```

---

## Card 21

**Front:** What is `/var/run/docker.sock`?

**Back:** It is the Unix socket commonly used by the Docker CLI and other local clients to communicate with the Docker daemon on Linux.

---

## Card 22

**Front:** `docker ps` gives permission denied, but `sudo docker ps` works. What does that suggest?

**Back:** Docker itself is probably running, while the current user likely lacks permission to access the Docker socket.

Check:

```bash
ls -l /var/run/docker.sock
groups
id
```

---

## Card 23

**Front:** What command can add the current user to the `docker` group when that access is appropriate?

**Back:**

```bash
sudo usermod -aG docker $USER
```

Then start a new login session before testing again.

**Security note:** Docker-group membership provides highly privileged access to the host.

---

## Card 24

**Front:** What commands are useful for debugging a Docker container?

**Back:**

```bash
sudo docker ps
sudo docker ps -a
sudo docker logs apache-container
sudo docker inspect apache-container
sudo docker port apache-container
```

---

## Card 25

**Front:** Why can a container stop immediately after starting?

**Back:** A common reason is that its main process exited. Check the container status and logs, then verify that the application runs as the foreground process.

```bash
sudo docker ps -a
sudo docker logs <container>
```

---

## Card 26

**Front:** If `curl localhost:81` works on EC2 but the browser cannot reach the public address, which boundary should you investigate next?

**Back:** Cloud/network access to the EC2 host, especially the applicable AWS Security Group rule and other host/network controls. A successful local curl shows that the host-to-container path is working.

---

## Card 27

**Front:** Why is cleanup normally `stop -> rm container -> rmi image`?

**Back:** First stop the running process, then remove the container object that references the image, and finally remove the image after its dependent container is gone.

```text
Running container
      ↓
STOP
      ↓
REMOVE CONTAINER
      ↓
REMOVE IMAGE
```

---

## Card 28

**Front:** What commands performed Day-25 cleanup?

**Back:**

```bash
sudo docker stop apache-container
sudo docker rm apache-container
sudo docker rmi custom-apache:v1
```

---

## Card 29

**Front:** What is the main problem with repeatedly configuring EC2 servers manually?

**Back:** Manual setup can produce configuration drift because different servers can end up with different packages, versions, files, permissions, and configuration.

Containers instead package the runtime into a versioned image.

---

## Card 30

**Front:** How does containerization improve releases compared with manual server configuration?

**Back:** The application and runtime requirements are encoded into a Dockerfile and built into a versioned image, giving teams a repeatable artifact that can move through development, testing, staging, and production.

**Memory hook:** `Configure servers -> Build artifacts`

---

## Card 31

**Front:** How do containers connect to CI/CD?

**Back:** CI/CD can automatically build, test, tag, publish, deploy, and verify versioned container images.

```text
Git Commit
   ↓
Build Image
   ↓
Test
   ↓
Tag
   ↓
Push
   ↓
Deploy
   ↓
Verify
   ↓
Retire Old Version
```

---

## Card 32

**Front:** Why deploy a new image version instead of manually modifying a running production container?

**Back:** Versioned images make deployments repeatable and auditable and allow the previous known image version to remain available for rollback. Manual changes create drift between the running system and the declared build artifact.

---

# Six Interview Flashcards

## Card 33

**Front:** Explain manual EC2 configuration versus containerized releases in one sentence.

**Back:** Manual EC2 deployment repeatedly configures servers, while containerized deployment builds repeatable, versioned images that can be deployed consistently.

---

## Card 34

**Front:** Explain local, host, and guest in one sentence.

**Back:** Local is my laptop, host is the EC2 machine running Docker Engine, and guest is the Docker container running Apache.

---

## Card 35

**Front:** Explain Docker socket permission troubleshooting in one sentence.

**Back:** Compare `docker ps` with `sudo docker ps`; if sudo works, inspect `/var/run/docker.sock` ownership and user groups before changing permissions.

---

## Card 36

**Front:** Explain image versus container and foreground process in one sentence.

**Back:** An image is a reusable template, a container is its running instance, and the container stays alive while its main foreground process stays alive.

---

## Card 37

**Front:** Explain the Day-25 Dockerfile and `-p 81:80` in one sentence.

**Back:** The Dockerfile starts from Ubuntu, installs Apache, copies the webpage, documents port 80, starts Apache in foreground mode, and `-p 81:80` forwards EC2 host port 81 to Apache on container port 80.

---

## Card 38

**Front:** Explain teardown and CI/CD in one sentence.

**Back:** Stop the running container, remove the container, then remove its unused image; CI/CD applies the same immutable-release idea by building and deploying new image versions instead of changing production manually.

---

# Rapid Recall Cards

## Card 39

**Front:** `FROM`?

**Back:** Choose the base image.

## Card 40

**Front:** `RUN`?

**Back:** Execute build-time commands.

## Card 41

**Front:** `COPY`?

**Back:** Copy files into the image.

## Card 42

**Front:** `EXPOSE`?

**Back:** Document the intended container port.

## Card 43

**Front:** `CMD`?

**Back:** Define the default container process.

## Card 44

**Front:** `81:80`?

**Back:** Host `81` -> container `80`.

## Card 45

**Front:** Image?

**Back:** Template.

## Card 46

**Front:** Container?

**Back:** Running instance.

## Card 47

**Front:** Main process exits?

**Back:** Container exits.

## Card 48

**Front:** Cleanup order?

**Back:** Stop -> remove container -> remove image.

---

# Final Memory Map

```text
Dockerfile
    ↓
BUILD
    ↓
Image
    ↓
RUN
    ↓
Container
    ↓
PUBLISH PORT
81 -> 80
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
CI/CD AUTOMATION
```
