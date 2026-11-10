# Day 26: Docker Containers Flashcards

Use these cards for quick revision. Read the **Front**, answer aloud, and then check the **Back**.

---

## Card 1

**Front:** What is the five-stage containerization lifecycle?

**Back:**

```text
Code/Dockerfile -> Build -> Image -> Run as Container -> Registry
```

The Dockerfile defines the environment, the build creates an image, the image is a reusable template, `docker run` creates a container, and the registry stores and distributes the image.

---

## Card 2

**Front:** What is a Docker image?

**Back:** A Docker image is a read-only, versioned template containing the application, runtime, dependencies, configuration, and filesystem layers needed to create containers.

---

## Card 3

**Front:** What is a Docker container?

**Back:** A container is a running or stopped instance of an image. It adds a writable container layer and runs with isolated processes, filesystem, and networking.

---

## Card 4

**Front:** What is the main difference between an image and a container?

**Back:** An image is the reusable template. A container is an instance created from that image with its own writable layer.

---

## Card 5

**Front:** What command builds the Day 26 version 1 image?

**Back:**

```bash
docker build -t day26-webserver:v1 .
```

`-t` sets the repository name and tag. The final `.` selects the current directory as the build context.

---

## Card 6

**Front:** What does `day26-webserver:v1` mean?

**Back:** `day26-webserver` is the repository or image name, and `v1` is the version tag.

---

## Card 7

**Front:** What is foreground container execution?

**Back:** The container's main process remains attached to the current terminal. Foreground mode is useful for direct observation, testing, and debugging.

```bash
docker run nginx:alpine
```

---

## Card 8

**Front:** What is detached container execution?

**Back:** Detached mode uses `-d` to run the container in the background and immediately return control of the terminal.

```bash
docker run -d nginx:alpine
```

It is suitable for long-running services such as web servers and APIs.

---

## Card 9

**Front:** How do you view running containers?

**Back:**

```bash
docker ps
```

Use `docker ps -a` to include stopped containers.

---

## Card 10

**Front:** How do you view the logs of a detached container?

**Back:**

```bash
docker logs day26-container
```

Follow the logs continuously with:

```bash
docker logs -f day26-container
```

---

## Card 11

**Front:** What does `-p 81:80` mean?

**Back:** It maps host port `81` to container port `80`.

```text
localhost:81 -> Docker host port 81 -> Container port 80
```

---

## Card 12

**Front:** Which port comes first in Docker port mapping?

**Back:** The host port comes first.

```text
-p HOST_PORT:CONTAINER_PORT
```

---

## Card 13

**Front:** How do you run the Day 26 container?

**Back:**

```bash
docker run -d --name day26-container -p 81:80 day26-webserver:v1
```

This runs the container in detached mode and maps host port 81 to container port 80.

---

## Card 14

**Front:** How do you access the Day 26 web page?

**Back:** Open:

```text
http://localhost:81
```

Or test it with:

```bash
curl http://localhost:81
```

---

## Card 15

**Front:** Does `EXPOSE 80` publish port 80 on the host?

**Back:** No. `EXPOSE 80` documents the intended container port. Use `-p`, such as `-p 81:80`, to publish and map the port on the host.

---

## Card 16

**Front:** What does `docker exec` do?

**Back:** It runs an additional command inside an already running container.

```bash
docker exec day26-container COMMAND
```

---

## Card 17

**Front:** What does `docker exec -it day26-container sh` do?

**Back:** It opens an interactive shell inside the running `day26-container` container.

- `-i`: keeps standard input open
- `-t`: allocates a terminal
- `sh`: starts a shell

---

## Card 18

**Front:** What is the difference between `docker run` and `docker exec`?

**Back:** `docker run` creates and starts a new container from an image. `docker exec` runs a command inside an existing running container.

---

## Card 19

**Front:** How is the Docker host different from the container?

**Back:** The host is the machine running Docker Engine. The container is an isolated application environment with its own filesystem view, processes, networking context, and writable layer. Containers share the host kernel.

---

## Card 20

**Front:** Where were the Day 26 Nginx webpage files stored inside the container?

**Back:**

```text
/usr/share/nginx/html/index.html
```

---

## Card 21

**Front:** Does changing a file inside a running container modify the original image?

**Back:** No. The change is written to the container's writable layer. The original image remains unchanged.

---

## Card 22

**Front:** Why is `docker commit` used in the Day 26 lab?

**Back:** It captures the modified running container's filesystem state as a new reusable image.

---

## Card 23

**Front:** What command commits the modified Day 26 container as version 2?

**Back:**

```bash
docker commit day26-container day26-webserver:v2
```

---

## Card 24

**Front:** Does `docker commit` overwrite version 1?

**Back:** No. It creates a separate image, `day26-webserver:v2`. The original `day26-webserver:v1` remains available.

---

## Card 25

**Front:** Why is rebuilding from a Dockerfile preferred over `docker commit` in production?

**Back:** A Dockerfile-based build is repeatable, reviewable, auditable, and suitable for automation. A manual container modification can be difficult to reproduce.

---

## Card 26

**Front:** How do you verify that both image versions exist?

**Back:**

```bash
docker images day26-webserver
```

The output should include `v1` and `v2`.

---

## Card 27

**Front:** How do you prove that image version 2 contains the modification?

**Back:** Remove or stop the old container, start a fresh container from `v2`, and verify the web page:

```bash
docker run -d --name day26-v2 -p 81:80 day26-webserver:v2
curl http://localhost:81
```

---

## Card 28

**Front:** What is the Docker Hub image naming format?

**Back:**

```text
USERNAME/REPOSITORY:TAG
```

Example:

```text
wilson123/day26-webserver:v2
```

---

## Card 29

**Front:** How do you authenticate to Docker Hub from the terminal?

**Back:**

```bash
docker login
```

Use the correct Docker Hub username and password or personal access token.

---

## Card 30

**Front:** How do you tag the local version 2 image for Docker Hub?

**Back:**

```bash
docker tag day26-webserver:v2 YOUR_USERNAME/day26-webserver:v2
```

---

## Card 31

**Front:** How do you push the Day 26 image to Docker Hub?

**Back:**

```bash
docker push YOUR_USERNAME/day26-webserver:v2
```

---

## Card 32

**Front:** Where do you verify a pushed image in Docker Hub?

**Back:** Sign in to Docker Hub, open **My Hub** or **Repositories**, select `day26-webserver`, open **Tags**, and confirm that `v2` exists.

---

## Card 33

**Front:** How do you download the published image from Docker Hub?

**Back:**

```bash
docker pull YOUR_USERNAME/day26-webserver:v2
```

---

## Card 34

**Front:** What does the Docker tag `latest` mean?

**Back:** `latest` is only a default tag name. It does not guarantee that the image is the newest, safest, or most appropriate version.

---

## Card 35

**Front:** What could cause `requested access to the resource is denied` during `docker push`?

**Back:** Common causes include:

- Logged in with the wrong Docker Hub account
- Image name does not begin with the correct username or organization
- Repository name is incorrect
- Account lacks permission to push to the namespace

---

## Card 36

**Front:** Can two containers both map to host port 81 at the same time?

**Back:** Normally no. Only one process or container can bind to the same host IP and port combination. Use another host port, such as `-p 82:80`.

---

## Card 37

**Front:** Does `docker stop` delete a container?

**Back:** No. It stops the container's main process. Remove the stopped container with:

```bash
docker rm CONTAINER_NAME
```

---

## Card 38

**Front:** Does removing a container remove its image?

**Back:** No. Containers and images are separate Docker objects. An image is removed separately with `docker rmi` when it is no longer needed.

---

## Card 39

**Front:** What is the difference between `docker exec` and `docker attach`?

**Back:** `docker exec` starts a new command inside the running container. `docker attach` connects the terminal to the container's existing main process.

---

## Card 40

**Front:** Summarize the complete Day 26 workflow.

**Back:**

```text
Create Dockerfile and index.html
-> Build image v1
-> Run container with 81:80 mapping
-> Modify the page using docker exec
-> Commit the container as image v2
-> Tag v2 with the Docker Hub username
-> Push v2
-> Verify the v2 tag in Docker Hub
```

---

# Command Recall Cards

## Card 41

**Front:** Build command?

**Back:**

```bash
docker build -t day26-webserver:v1 .
```

---

## Card 42

**Front:** Run command?

**Back:**

```bash
docker run -d --name day26-container -p 81:80 day26-webserver:v1
```

---

## Card 43

**Front:** Enter container command?

**Back:**

```bash
docker exec -it day26-container sh
```

---

## Card 44

**Front:** Commit command?

**Back:**

```bash
docker commit day26-container day26-webserver:v2
```

---

## Card 45

**Front:** Docker Hub tag command?

**Back:**

```bash
docker tag day26-webserver:v2 YOUR_USERNAME/day26-webserver:v2
```

---

## Card 46

**Front:** Docker Hub push command?

**Back:**

```bash
docker login
docker push YOUR_USERNAME/day26-webserver:v2
```

---

# Final Memory Card

**Front:** What is the most important Day 26 concept?

**Back:**

```text
An image is the immutable template.
A container is its runtime instance with a writable layer.
A commit captures that modified layer as a new image.
A registry stores and distributes the tagged image.
```
