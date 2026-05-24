# Docker Container: Detailed Technical Report

## Introduction

A **Docker container** is a lightweight, isolated runtime environment used to package and run applications together with all required dependencies.

Docker containers solve the classic problem:

```text
"It works on my machine."
```

A container includes:

* Application code
* Runtime
* System libraries
* Dependencies
* Environment variables
* Configuration

Containers run consistently across:

* Local machines
* VPS servers
* Cloud infrastructure
* CI/CD pipelines
* Kubernetes clusters

Official website: [Docker](https://www.docker.com)

---

# What Docker Is

Docker is:

* A containerization platform
* A runtime engine
* A packaging system
* A deployment platform

Docker allows developers to:

* Build applications
* Package them into containers
* Deploy them consistently anywhere

---

# What a Docker Container Is

A Docker container is:

* An isolated process
* Running from a Docker image
* Sharing the host OS kernel
* With its own filesystem, networking, and processes

Container characteristics:

* Lightweight
* Fast startup
* Portable
* Reproducible
* Ephemeral by default

---

# Container vs Virtual Machine

| Feature        | Docker Container | Virtual Machine |
| -------------- | ---------------- | --------------- |
| Boot Time      | Seconds          | Minutes         |
| Size           | MBs              | GBs             |
| OS Included    | No               | Yes             |
| Performance    | Near-native      | Lower           |
| Isolation      | Process-level    | Hardware-level  |
| Resource Usage | Low              | High            |

---

# Docker Architecture

Core components:

```text
Developer
    ↓
Docker CLI
    ↓
Docker Engine
    ↓
Containers
```

Main components:

* Docker CLI
* Docker Daemon
* Docker Engine
* Docker Images
* Docker Containers
* Docker Registry

---

# Docker Engine

Docker Engine is the runtime responsible for:

* Building images
* Running containers
* Managing networking
* Managing storage
* Handling container lifecycle

---

# Docker Image

A Docker image is:

* A read-only template
* Used to create containers

Analogy:

```text
Image = Blueprint
Container = Running Instance
```

---

# Docker Container Lifecycle

```text
Build → Run → Stop → Remove
```

Commands:

## Build

```bash id="7pr12l"
docker build -t myapp .
```

## Run

```bash id="rwkryx"
docker run myapp
```

## Stop

```bash id="dgnffy"
docker stop container_id
```

## Remove

```bash id="jlwmzr"
docker rm container_id
```

---

# Installing Docker

## Linux

Official install guide:
[Docker Engine Install Docs](https://docs.docker.com/engine/install/?utm_source=ovishekh.com)

---

## macOS

Install:

* Docker Desktop

Official:
[Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/?utm_source=ovishekh.com)

---

# Basic Docker Commands

## Check Version

```bash id="bkjh5t"
docker --version
```

---

## Pull Image

```bash id="gl3iwi"
docker pull nginx
```

Downloads image from Docker Hub.

---

## Run Container

```bash id="mrjlwm"
docker run nginx
```

---

## Run Detached

```bash id="2q9s1y"
docker run -d nginx
```

`-d`:

* Runs in background

---

## Port Mapping

```bash id="blj3v4"
docker run -p 8080:80 nginx
```

Meaning:

```text
Host Port 8080 → Container Port 80
```

---

## List Containers

```bash id="5yj7is"
docker ps
```

---

## List All Containers

```bash id="x0q7nq"
docker ps -a
```

---

## View Logs

```bash id="6h1c8g"
docker logs container_id
```

---

## Open Shell

```bash id="h6p09j"
docker exec -it container_id bash
```

---

# Dockerfile

A **Dockerfile** defines how to build an image.

Example:

```dockerfile id="q2d1lr"
FROM node:20

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

---

# Dockerfile Instructions

## FROM

Base image.

```dockerfile id="kn2dpr"
FROM ubuntu
```

---

## WORKDIR

Sets working directory.

```dockerfile id="ezg9n2"
WORKDIR /app
```

---

## COPY

Copies files.

```dockerfile id="7lv0cr"
COPY . .
```

---

## RUN

Executes command during build.

```dockerfile id="4i9bb0"
RUN npm install
```

---

## CMD

Default startup command.

```dockerfile id="6tltx3"
CMD ["node", "server.js"]
```

---

# Building Images

```bash id="5o2x0x"
docker build -t myapp .
```

Explanation:

* `build` → create image
* `-t` → tag/name image
* `.` → current directory

---

# Running Applications

## Node.js Example

```bash id="2o1og4"
docker run -p 3000:3000 myapp
```

---

# Container Networking

Containers communicate through Docker networks.

## Create Network

```bash id="mz9q0t"
docker network create mynetwork
```

---

## Run on Network

```bash id="0k4pma"
docker run --network mynetwork nginx
```

---

# Volumes

Containers are ephemeral.

Volumes persist data.

## Create Volume

```bash id="c56v5v"
docker volume create mydata
```

---

## Attach Volume

```bash id="df9m9x"
docker run -v mydata:/data nginx
```

---

# Bind Mounts

Maps local directories into container.

```bash id="ap0q2f"
docker run -v $(pwd):/app node
```

Useful for:

* Development
* Live code changes

---

# Environment Variables

```bash id="26j2z6"
docker run -e NODE_ENV=production myapp
```

---

# Docker Compose

Docker Compose manages multi-container applications.

Official docs:
[Docker Compose Documentation](https://docs.docker.com/compose/?utm_source=ovishekh.com)

---

# docker-compose.yml Example

```yaml id="7i9m7k"
services:

  app:
    build: .
    ports:
      - "3000:3000"

  redis:
    image: redis

  db:
    image: postgres
```

---

# Starting Compose

```bash id="djlwmv"
docker compose up
```

---

# Background Mode

```bash id="gyty5l"
docker compose up -d
```

---

# Stopping Compose

```bash id="x36zq3"
docker compose down
```

---

# Container Isolation

Containers isolate:

* Processes
* Filesystems
* Networks
* Dependencies

But containers:

* Share the host kernel

Unlike VMs:

* Containers do not emulate hardware

---

# Resource Limits

## CPU Limit

```bash id="t7p25t"
docker run --cpus="2" nginx
```

---

## Memory Limit

```bash id="sjjlwm"
docker run -m 512m nginx
```

---

# Docker Registry

A registry stores Docker images.

Examples:

* Docker Hub
* GitHub Container Registry
* AWS ECR
* Google Artifact Registry

---

# Docker Hub

Default public registry.

Official:
[Docker Hub](https://hub.docker.com?utm_source=ovishekh.com)

---

# Push Image

```bash id="v9y15h"
docker push username/myapp
```

---

# Pull Image

```bash id="y3i4my"
docker pull username/myapp
```

---

# Multi Container Architecture

Example stack:

```text
Internet
   ↓
Caddy/Nginx
   ↓
Frontend Container
   ↓
Backend API Container
   ↓
Database Container
```

---

# Example Production Setup

## Next.js + PostgreSQL + Redis

```yaml id="4dwnhx"
services:

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"

  backend:
    build: ./backend
    ports:
      - "4000:4000"

  postgres:
    image: postgres

  redis:
    image: redis
```

---

# Docker and Caddy Together

Common architecture:

```text
Internet
   ↓
Caddy Container
   ↓
Application Containers
```

Example Compose:

```yaml id="g1hvsu"
services:

  caddy:
    image: caddy

  app:
    build: .
```

---

# Docker Security

Best practices:

* Use minimal base images
* Avoid running as root
* Scan images
* Use trusted registries
* Keep images updated

---

# Alpine Images

Lightweight Linux image.

Example:

```dockerfile id="mj2fut"
FROM node:20-alpine
```

Benefits:

* Smaller image size
* Faster deployment

---

# Layer System

Docker images use layers.

Example:

```dockerfile id="2px08y"
FROM ubuntu
RUN apt install nginx
COPY . .
```

Each instruction creates:

* A cached layer

Benefits:

* Faster rebuilds
* Reduced bandwidth

---

# Container Orchestration

For large scale deployments:

* Kubernetes
* Docker Swarm
* Nomad

Docker handles:

* Single host containerization

Kubernetes handles:

* Cluster orchestration

---

# Common Use Cases

Docker is used for:

* Web applications
* APIs
* AI inference servers
* CI/CD
* Databases
* Dev environments
* Microservices
* VPS deployments

---

# Advantages of Docker

## Consistency

Same environment everywhere.

---

## Fast Deployment

Containers start quickly.

---

## Scalability

Easy horizontal scaling.

---

## Isolation

Applications do not conflict.

---

## Reproducibility

Infrastructure becomes deterministic.

---

# Limitations

## Shared Kernel

Containers are less isolated than VMs.

---

## Persistence Complexity

Stateful apps require volume management.

---

## Learning Curve

Networking and orchestration become complex at scale.

---

# Common Docker Problems

## Port Conflict

Error:

```text id="vfowkp"
bind: address already in use
```

Cause:

* Another service already uses the port.

---

## Large Image Sizes

Cause:

* Unoptimized Dockerfiles
* Large dependencies

---

## Container Exits Immediately

Cause:

* Main process terminates

---

## Permission Errors

Cause:

* Incorrect filesystem ownership
* Non-root execution conflicts

---

# Docker Best Practices

## Use `.dockerignore`

Example:

```text id="wr9ypq"
node_modules
.git
.env
```

---

## Use Multi-stage Builds

Example:

```dockerfile id="2v01zq"
FROM node:20 AS builder

FROM nginx
```

Benefits:

* Smaller production images

---

## Pin Versions

Bad:

```dockerfile id="8mjlwm"
FROM node:latest
```

Good:

```dockerfile id="iwd0mv"
FROM node:20.11
```

---

# Enterprise Usage

Docker is heavily used in:

* AWS
* Google Cloud
* Azure
* Kubernetes ecosystems
* SaaS platforms
* AI infrastructure
* DevOps pipelines

---

# Docker vs Podman

| Feature             | Docker    | Podman   |
| ------------------- | --------- | -------- |
| Daemon Required     | Yes       | No       |
| Ecosystem           | Larger    | Growing  |
| Rootless Support    | Partial   | Strong   |
| Enterprise Adoption | Very High | Moderate |

---

# Docker Internals

Docker relies on Linux kernel technologies:

* Namespaces
* cgroups
* OverlayFS
* Union filesystems

These provide:

* Isolation
* Resource control
* Layered filesystems

---

# Minimal Example

## Dockerfile

```dockerfile id="k0v1t7"
FROM nginx
```

## Build

```bash id="mjlwmk"
docker build -t test .
```

## Run

```bash id="hyq3h2"
docker run -p 8080:80 test
```

Result:

```text id="o5mtnl"
localhost:8080 → nginx server
```

---

# Official Documentation

## Docker Docs

[Docker Documentation](https://docs.docker.com/?utm_source=ovishekh.com)

## Dockerfile Reference

[Dockerfile Reference](https://docs.docker.com/reference/dockerfile/?utm_source=ovishekh.com)

## Docker Compose

[Docker Compose Docs](https://docs.docker.com/compose/?utm_source=ovvishekh.com)

## Docker Hub

[Docker Hub](https://hub.docker.com?utm_source=ovvishekh.com)

---

# Summary

A Docker container is:

* An isolated runtime environment
* Built from Docker images
* Lightweight and portable
* Used for reproducible deployments

Docker enables:

* Consistent environments
* Faster deployments
* Infrastructure portability
* Scalable application architecture

Core workflow:

```text id="vh6l7j"
Dockerfile
    ↓
Docker Image
    ↓
Docker Container
    ↓
Deployment
```
