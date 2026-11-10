# Day 26: Build, Modify, Commit, and Push a Custom Web Server Image

## Objective

Complete the Docker container lifecycle:

1. Create a custom web server image.
2. Build and tag the image.
3. Run a container with port mapping.
4. Modify the running container.
5. Commit the modified container as a new image.
6. Push the new image to Docker Hub.
7. Explain the difference between an image, a running container, and a committed image.

---

## Prerequisites

- Docker installed and running.
- A terminal or command prompt.
- A Docker Hub account.
- Internet access for pulling the Nginx base image and pushing the final image.

Verify Docker:

```bash
docker --version
docker ps
```

If Docker is working, continue with the lab.

---

## Step 1: Create the project directory

```bash
mkdir day26-webserver
cd day26-webserver
```

Confirm the current directory:

```bash
pwd
```

---

## Step 2: Create the web page

Create `index.html`:

```bash
cat > index.html <<'HTML'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FDE Day 26</title>
</head>
<body>
    <h1>Hello from FDE Day 26!</h1>
    <p>This page is running inside a Docker container.</p>
</body>
</html>
HTML
```

Verify the file:

```bash
cat index.html
```

---

## Step 3: Create the Dockerfile

Create a file named `Dockerfile`:

```bash
cat > Dockerfile <<'DOCKERFILE'
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
DOCKERFILE
```

Verify the Dockerfile:

```bash
cat Dockerfile
```

The project should contain:

```text
day26-webserver/
├── Dockerfile
└── index.html
```

Check with:

```bash
ls -la
```

### Dockerfile explanation

- `FROM nginx:alpine` uses the lightweight Alpine-based Nginx image.
- `COPY` adds the custom page to Nginx's default web root.
- `EXPOSE 80` documents that the web server listens on container port 80.

---

## Step 4: Build and tag image version 1

Build the image:

```bash
docker build -t day26-webserver:v1 .
```

Breaking down the command:

- `docker build` builds an image.
- `-t` assigns a repository name and tag.
- `day26-webserver` is the repository name.
- `v1` is the version tag.
- `.` tells Docker to use the current directory as the build context.

Verify the image:

```bash
docker images day26-webserver
```

Expected result:

```text
REPOSITORY          TAG
 day26-webserver     v1
```

---

## Step 5: Run the container with port mapping

Run the container in detached mode:

```bash
docker run -d --name day26-container -p 81:80 day26-webserver:v1
```

Command explanation:

- `-d` runs the container in the background.
- `--name day26-container` gives the container a readable name.
- `-p 81:80` maps host port 81 to container port 80.
- `day26-webserver:v1` is the image used to create the container.

Verify the running container:

```bash
docker ps
```

Look for a port mapping similar to:

```text
0.0.0.0:81->80/tcp
```

Test from the terminal:

```bash
curl http://localhost:81
```

Alternatively, open this address in a browser:

```text
http://localhost:81
```

Expected page content:

```text
Hello from FDE Day 26!
This page is running inside a Docker container.
```

---

## Step 6: Enter and modify the running container

Open an interactive shell inside the container:

```bash
docker exec -it day26-container sh
```

Display the current page from inside the container:

```sh
cat /usr/share/nginx/html/index.html
```

Replace it with a modified page:

```sh
cat > /usr/share/nginx/html/index.html <<'HTML'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FDE Day 26 Modified</title>
</head>
<body>
    <h1>Day 26: Live Container Modified!</h1>
    <p>This page was changed inside the running container using docker exec.</p>
</body>
</html>
HTML
```

Verify the modification:

```sh
cat /usr/share/nginx/html/index.html
```

Exit the container shell:

```sh
exit
```

Verify the live change:

```bash
curl http://localhost:81
```

Refresh `http://localhost:81` in the browser. The modified page should appear immediately.

> Important: This change exists in the running container's writable layer. The original `day26-webserver:v1` image has not changed.

---

## Step 7: Commit the modified container as version 2

Capture the current container state as a new image:

```bash
docker commit day26-container day26-webserver:v2
```

Verify both image versions:

```bash
docker images day26-webserver
```

Expected result:

```text
REPOSITORY          TAG
 day26-webserver     v2
 day26-webserver     v1
```

---

## Step 8: Prove that version 2 contains the modification

Stop and remove the original running container:

```bash
docker stop day26-container
docker rm day26-container
```

Start a fresh container from `v2`:

```bash
docker run -d --name day26-v2 -p 81:80 day26-webserver:v2
```

Verify it:

```bash
docker ps
curl http://localhost:81
```

The modified page should still appear. This proves that the live change was captured in `day26-webserver:v2`.

---

## Step 9: Log in to Docker Hub

Authenticate from the terminal:

```bash
docker login
```

Enter the Docker Hub username and password or personal access token when prompted.

Verify the logged-in username if needed:

```bash
docker info
```

Do not place passwords or access tokens in commands, screenshots, Markdown files, or Git repositories.

---

## Step 10: Tag the image for Docker Hub

Docker Hub image names use this format:

```text
DOCKERHUB_USERNAME/REPOSITORY:TAG
```

Replace `YOUR_DOCKERHUB_USERNAME` with the actual Docker Hub username:

```bash
docker tag day26-webserver:v2 YOUR_DOCKERHUB_USERNAME/day26-webserver:v2
```

Example only:

```bash
docker tag day26-webserver:v2 wilson123/day26-webserver:v2
```

Verify the new tag:

```bash
docker images
```

The local image list should include:

```text
YOUR_DOCKERHUB_USERNAME/day26-webserver   v2
```

---

## Step 11: Push the image to Docker Hub

Push the tagged image:

```bash
docker push YOUR_DOCKERHUB_USERNAME/day26-webserver:v2
```

A successful push shows uploaded layers and a digest.

### Common push error

If Docker reports `requested access to the resource is denied`, check:

1. `docker login` used the correct account.
2. The image name begins with the same Docker Hub username.
3. The repository/tag spelling is correct.
4. The Docker Hub account has permission to push to that namespace.

---

## Step 12: Verify the image in Docker Hub

1. Sign in to Docker Hub using the same account used by `docker login`.
2. Open **My Hub** or **Repositories**.
3. Select the `day26-webserver` repository.
4. Open the **Tags** section.
5. Confirm that the `v2` tag is present.

The full image reference should be:

```text
YOUR_DOCKERHUB_USERNAME/day26-webserver:v2
```

Optionally prove that the registry image works by removing the local tagged version and pulling it again:

```bash
docker pull YOUR_DOCKERHUB_USERNAME/day26-webserver:v2
```

Run the pulled image on another host port:

```bash
docker run -d --name day26-pulled -p 82:80 YOUR_DOCKERHUB_USERNAME/day26-webserver:v2
```

Test it:

```bash
curl http://localhost:82
```

---

## Step 13: Conceptual explanation

### Original image: `day26-webserver:v1`

The original image is the read-only template produced from the Dockerfile and original `index.html`.

### Modified running container: `day26-container`

The running container is an instance of `v1`. The change made with `docker exec` was stored in the container's writable layer and did not modify `v1`.

### Committed image: `day26-webserver:v2`

The committed image captures the modified container filesystem as a new reusable image. New containers created from `v2` include the modified page.

### Lifecycle summary

```text
Dockerfile + index.html
        |
        | docker build
        v
 day26-webserver:v1
        |
        | docker run
        v
  Running container
        |
        | docker exec and modify
        v
 Modified container
        |
        | docker commit
        v
 day26-webserver:v2
        |
        | docker tag + docker push
        v
      Docker Hub
```

> Production note: `docker commit` is useful for this lifecycle exercise and for investigation snapshots. For repeatable production delivery, update the source or Dockerfile and rebuild the image instead of relying on manual container changes.

---

## Interview review

### 1. What is the five-stage containerization assembly line?

A concise answer is:

```text
Code/Dockerfile -> Build -> Image -> Run as Container -> Push to Registry
```

- Code and Dockerfile define the application environment.
- Build creates the image.
- The image is the reusable template.
- Run creates a container instance.
- A registry stores and distributes tagged images.

### 2. Foreground vs detached execution

Foreground:

```bash
docker run nginx:alpine
```

The process remains attached to the terminal. This is useful for direct output and debugging.

Detached:

```bash
docker run -d nginx:alpine
```

The container runs in the background. This is suitable for long-running services.

### 3. What does `-p 81:80` mean?

```text
Host port 81 -> Container port 80
```

A request to `http://localhost:81` is forwarded to the service listening on port 80 inside the container.

### 4. What does `docker exec -it` do?

It executes an interactive command inside an already running container:

```bash
docker exec -it day26-container sh
```

- `-i` keeps standard input open.
- `-t` allocates a terminal.
- The resulting shell operates in the container environment, not directly on the host.

### 5. Why use `docker commit` after modifying a container?

The modification is in the container's writable layer. `docker commit` captures that state as a new image so it can be reused. It does not overwrite the original image.

### 6. What is the image tag format and how is it pushed?

Format:

```text
username/repository:tag
```

Commands:

```bash
docker login
docker tag day26-webserver:v2 YOUR_DOCKERHUB_USERNAME/day26-webserver:v2
docker push YOUR_DOCKERHUB_USERNAME/day26-webserver:v2
```

---

## Evidence checklist

Capture the following for the lab submission:

- [ ] `Dockerfile` content.
- [ ] Original `index.html` content.
- [ ] Successful `docker build` output.
- [ ] `docker images` showing `day26-webserver:v1`.
- [ ] `docker ps` showing `81:80` port mapping.
- [ ] Browser or `curl` output showing the original page.
- [ ] `docker exec -it` session and modified page.
- [ ] Browser or `curl` output showing the live modification.
- [ ] Successful `docker commit` command.
- [ ] `docker images` showing both `v1` and `v2`.
- [ ] Fresh container created from `v2` showing the modified page.
- [ ] Successful Docker Hub push output.
- [ ] Docker Hub repository **Tags** page showing `v2`.

---

## Cleanup commands

Run these only after collecting the required evidence:

```bash
docker stop day26-v2 2>/dev/null || true
docker rm day26-v2 2>/dev/null || true
docker stop day26-pulled 2>/dev/null || true
docker rm day26-pulled 2>/dev/null || true
```

Optional local image cleanup:

```bash
docker rmi day26-webserver:v1
docker rmi day26-webserver:v2
docker rmi YOUR_DOCKERHUB_USERNAME/day26-webserver:v2
```

Do not remove the images until the lab has been verified and all screenshots have been captured.

---

## Final result

At the end of Day 26, the following should be true:

- A custom Nginx Docker image was built as `day26-webserver:v1`.
- A container was exposed through host port 81.
- The running container's web page was modified using `docker exec -it`.
- The modified state was captured as `day26-webserver:v2`.
- The `v2` image was tagged and pushed to Docker Hub.
- The `v2` tag was verified in the Docker Hub repository.
