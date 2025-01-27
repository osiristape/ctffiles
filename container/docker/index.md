---
title: Docker Pentesting
description: Docker is a set of platform as a service products that use OS-level virtualization to deliver software in packages called containers.  Default ports are 2375, 2376.
tags:
    - Container
    - Privilege Escalation
refs:
date: 2023-10-11
draft: false
---

## Investigation

### Find Docker Binary

If we cannot use `docker` command by default, we need to find the docker binary.

```bash
find / -name "docker" 2>/dev/null
```

### Basic Commands

```sh
# List images
docker images
docker image ls
# The history of an image
docker image history <image-name>

# List containers running
docker container ls
# or
docker ps

# List all containers
docker container ls -a
# or
docker ps -a

# List secrets
docker secret ls

# Check configuration of container
docker inspect --format='{{json .Config}}' <container_id_or_name>

# Get a port which is used by the container
docker port <container_id_or_name>

# Scan vulnerabilies (CVEs)
docker scan cves <image>
docker scan cves alpine

# View the SBOM (Software Bill of Materials) for an image
# We can investigate vulnerabilities from the list of packages.
docker sbom alpine:latest
# Json format
docker sbom alpine:latest --format syft-json

# Spawn the shell in the container
docker exec -it <container_id> /bin/bash

# Kill the running docker container
docker kill <container_id>
```

### Check if Containers Running

In target machine, observe the network status by running `netstat` or `ss` command.

```sh
netstat -punta
# or
ss -ltu

# -------------------------------------------------------

tcp  0  0  127.0.0.1:2375  0.0.0.0:*  LISTEN  -
```

<br />

## Basic Operations

### Run a New Container

First check the docker images listed.

```sh
docker images
```

Then run a new container from the image.

```sh
# -d: detached mode (background)
# -p: map the port of the host to the port in the container
docker run -dp 80:80 <image-name>
```

If you want to run a new container from a remote repository, run the following.

```sh 
# --rm: Removes the anonymous volumes when the container is removed
# -i: interactive
# -t: tty
# --network=host: The container is not isolated from the Docker host. The IP address is your own home IP address.
docker run --rm -it --network=host <repository>/<image>

# /bin/bash: spawn a shell within the container
docker run -it nginx /bin/bash
```

### Start a Container which is stopped

```sh
# List all containers and check the target ID
docker container ls -a

# Start the container
docker container start <container-id>
```

### Run Commands in a Container

```sh
# List containers running and check the target container ID
docker ps

# Run commands by giving the container ID
docker exec <container-id> whoami
docker exec <container-id> cat sample.txt
```

### Stop a Container

```sh
# List running containers and check the target container ID
docker ps

# Stop the container by giving the ID
docker stop <container-id>
```

### Remove a Container

```sh
# List all containers and check the target container ID
docker ps -a

# Remove the container by givine the ID
docker rm <container-id>
# Force to remove the running container (-f)
docker rm -f <container-id>
```

### Build a Container Image

First off, create a **Dockerfile** in the root directory of the project.

```sh
FROM node:12-alpine

RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

Now run the following command to build the container image.  
This command uses the Dockerfile.

```sh
# -t: name a tag of the image
docker build -t <tag-name> .
```

### Scan a Container Image

```sh
docker scan <image-name>
```

### Pull a Docker Image

We need to download a docker image to start a container at first.

```bash
docker pull <image>
docker pull nginx

# Specify a tag
docker pull <image>:<tag>
docker pull nginx:latest
docker pull nginx:stable
```

### Remove a Docker Image

```sh
# List images and check the target image ID
docker images

# Remove the image by giving the ID
docker rmi <image-id>
```

### Publish a Docker Image

Before doing below, you need to sign up the Docker Hub and sign in, then create a new repository in your dashboard.

```sh
# Login
docker login -u <your-username>

# Tag a new image
docker tag <source-image> <your-username>/<target-image>

# Push
docker push <your-username>/<target-image>
```

