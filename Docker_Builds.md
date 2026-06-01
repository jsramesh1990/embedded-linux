# Docker Builds

## Table of Contents

* Introduction
* What is a Docker Build?
* Why Do We Need Docker Builds?
* Docker Build Architecture
* Dockerfile
* Docker Build Process
* Important Docker Build Commands
* Multi-Stage Builds
* Build Context
* Docker Build Cache
* Common Use Cases
* Advantages
* Disadvantages
* Best Practices
* Interview Questions
* Conclusion

---

# Introduction

Docker is a containerization platform that allows applications to run consistently across different environments.

Before running a container, we first need to create a Docker image. This process is called a **Docker Build**.

A Docker build takes source code, dependencies, configurations, and instructions from a Dockerfile and creates a portable Docker image.

---

# What is a Docker Build?

A Docker build is the process of creating a Docker image from a Dockerfile.

The Docker engine reads instructions from the Dockerfile and executes them step by step to generate an image.

```text
Source Code
     ↓
Dockerfile
     ↓
Docker Build
     ↓
Docker Image
     ↓
Docker Container
```

---

# Why Do We Need Docker Builds?

### Problem Without Docker

```text
Developer Machine
       ↓
Works Fine
       ↓
Production Server
       ↓
Fails Due To:
- Missing Libraries
- Different OS Versions
- Dependency Conflicts
```

### Solution Using Docker Build

```text
Developer Machine
       ↓
Docker Build
       ↓
Docker Image
       ↓
Same Image Everywhere
```

### Benefits

* Consistent environments
* Reproducible builds
* Dependency isolation
* Easy deployment
* Faster application delivery
* Platform portability

---

# Docker Build Architecture

```text
Application Code
        ↓
Dockerfile
        ↓
docker build
        ↓
Docker Engine
        ↓
Docker Image
        ↓
Docker Registry
        ↓
Docker Container
```

---

# Dockerfile

A Dockerfile is a text file containing instructions to build an image.

Example:

```dockerfile
FROM ubuntu:22.04

RUN apt-get update

RUN apt-get install -y nginx

COPY index.html /var/www/html/

CMD ["nginx", "-g", "daemon off;"]
```

---

# Common Dockerfile Instructions

## FROM

Defines the base image.

```dockerfile
FROM ubuntu:22.04
```

---

## RUN

Executes commands during build.

```dockerfile
RUN apt-get update
```

---

## COPY

Copies files into the image.

```dockerfile
COPY app.py /app/
```

---

## ADD

Copies files and supports URL extraction.

```dockerfile
ADD archive.tar.gz /app/
```

---

## WORKDIR

Sets the working directory.

```dockerfile
WORKDIR /app
```

---

## ENV

Defines environment variables.

```dockerfile
ENV APP_ENV=production
```

---

## EXPOSE

Documents container ports.

```dockerfile
EXPOSE 8080
```

---

## CMD

Default command when container starts.

```dockerfile
CMD ["python3", "app.py"]
```

---

## ENTRYPOINT

Defines the main executable.

```dockerfile
ENTRYPOINT ["python3"]
```

---

# Docker Build Process

## Step 1: Create Dockerfile

```dockerfile
FROM python:3.11

COPY . /app

WORKDIR /app

RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

---

## Step 2: Build Image

```bash
docker build -t myapp:v1 .
```

Where:

```text
-t      → Tag image
myapp   → Image name
v1      → Version tag
.       → Current directory (build context)
```

---

## Step 3: Verify Image

```bash
docker images
```

---

## Step 4: Run Container

```bash
docker run myapp:v1
```

---

# Important Docker Build Commands

## Build Image

```bash
docker build -t myimage:v1 .
```

---

## Build Using Specific Dockerfile

```bash
docker build -f Dockerfile.prod -t myimage:v1 .
```

---

## Build Without Cache

```bash
docker build --no-cache -t myimage:v1 .
```

---

## View Build History

```bash
docker history myimage:v1
```

---

## Remove Build Cache

```bash
docker builder prune
```

---

# Build Context

The build context is the set of files sent to Docker during build.

Example:

```bash
docker build .
```

Docker sends:

```text
Current Directory
     ↓
Docker Daemon
```

Large contexts increase build time.

Use:

```text
.dockerignore
```

to exclude unnecessary files.

Example:

```text
.git
node_modules
build/
logs/
```

---

# Docker Build Cache

Docker caches build layers.

Example:

```dockerfile
FROM ubuntu

RUN apt-get update

COPY app.py .
```

If `app.py` changes:

```text
FROM          → Cached
RUN           → Cached
COPY          → Rebuilt
```

Benefits:

* Faster builds
* Reduced resource usage

---

# Multi-Stage Builds

Used to reduce image size.

## Without Multi-Stage Build

```dockerfile
FROM golang:1.22

COPY . .

RUN go build app
```

Final image contains:

* Source code
* Compiler
* Build tools

Large image size.

---

## With Multi-Stage Build

```dockerfile
FROM golang:1.22 AS builder

WORKDIR /app

COPY . .

RUN go build -o app

FROM ubuntu:22.04

COPY --from=builder /app/app /app/app

CMD ["/app/app"]
```

Result:

```text
Final Image
    ↓
Only Application Binary
```

Smaller and more secure.

---

# Common Use Cases

## CI/CD Pipelines

Automatically build images on:

* Push
* Pull Request
* Release

---

## Microservices

Each service gets its own image.

```text
Frontend Image
Backend Image
Database Image
```

---

## Cloud Deployments

Used in:

* Kubernetes
* AWS ECS
* Azure Container Apps
* Google Kubernetes Engine

---

## Embedded Linux Development

Build:

* Development containers
* Yocto build environments
* SDK environments

---

# Advantages

## Portability

Runs consistently across environments.

---

## Reproducibility

Same image every time.

---

## Dependency Isolation

No host dependency conflicts.

---

## Faster Deployment

Pre-built images deploy quickly.

---

## CI/CD Friendly

Integrates with GitHub Actions, Jenkins, GitLab CI, etc.

---

## Version Control

Images can be tagged and managed.

---

# Disadvantages

## Large Images

Poor Dockerfiles create large images.

---

## Build Time

Complex applications may take longer.

---

## Security Risks

Outdated base images may contain vulnerabilities.

---

## Cache Issues

Stale caches can cause unexpected behavior.

---

# Best Practices

### Use Official Base Images

```dockerfile
FROM python:3.11-slim
```

---

### Minimize Layers

```dockerfile
RUN apt-get update && apt-get install -y curl
```

---

### Use Multi-Stage Builds

Reduces image size.

---

### Use .dockerignore

Avoid sending unnecessary files.

---

### Pin Image Versions

Good:

```dockerfile
FROM ubuntu:22.04
```

Bad:

```dockerfile
FROM ubuntu:latest
```

---

### Keep Images Small

Smaller images:

* Build faster
* Deploy faster
* Improve security

---

# Interview Questions

## What is a Docker Build?

A Docker build is the process of creating a Docker image from a Dockerfile.

---

## Why Do We Need Docker Builds?

Docker builds create consistent, portable, and reproducible environments that eliminate dependency conflicts and deployment issues.

---

## What is a Dockerfile?

A Dockerfile is a text file containing instructions used to build a Docker image.

---

## What is Build Context?

The set of files sent to the Docker daemon during image creation.

---

## What is Docker Cache?

Docker stores intermediate layers and reuses them to speed up future builds.

---

## What is a Multi-Stage Build?

A Docker build technique that uses multiple build stages to create smaller and more secure images.

---

## How Do You Build an Image?

```bash
docker build -t myimage:v1 .
```

---

## What is the Difference Between CMD and ENTRYPOINT?

| CMD               | ENTRYPOINT         |
| ----------------- | ------------------ |
| Default command   | Main executable    |
| Easily overridden | Harder to override |
| Optional          | Usually mandatory  |

---

# Most Asked Interview Question

### Why Do We Need Docker Builds?

**Answer:**

Docker builds are used to create portable and reproducible application images. They package the application, dependencies, libraries, and configurations into a single image that can run consistently across development, testing, and production environments. This eliminates dependency issues, improves deployment reliability, and supports modern CI/CD workflows.

---

# Conclusion

Docker builds are the foundation of containerized application deployment. They convert application code and dependencies into portable Docker images that can run consistently across any environment. By using Dockerfiles, build caching, and multi-stage builds, developers can create efficient, secure, and scalable container images for modern software development and deployment.
