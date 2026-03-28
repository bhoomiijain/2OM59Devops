# 1. Dockerfile & Image Building

## What is a Dockerfile?

A **Dockerfile** is a text file with a set of **instructions** to build a Docker image automatically.

> Think of it as a **recipe** — Docker reads it top to bottom and creates your image layer by layer.

---

## Build Context & .dockerignore

**Build context** = the folder you pass to `docker build` (everything in it gets sent to Docker daemon).

**.dockerignore** = like `.gitignore` — tells Docker which files/folders to **exclude** from the build context.

```
# .dockerignore example
node_modules
*.log
.env
__pycache__
```

---

## Core Dockerfile Instructions

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image to start from | `FROM ubuntu:22.04` |
| `RUN` | Execute a command during build | `RUN apt-get install -y python3` |
| `COPY` | Copy files from build context to image | `COPY app.py /app/` |
| `ADD` | Like COPY + can handle URLs & tar files | `ADD app.tar.gz /app/` |
| `CMD` | Default command when container starts | `CMD ["python3", "app.py"]` |
| `ENTRYPOINT` | Sets the main executable (cannot be overridden) | `ENTRYPOINT ["nginx"]` |
| `WORKDIR` | Set working directory inside container | `WORKDIR /app` |
| `ENV` | Set environment variable | `ENV PORT=8080` |
| `EXPOSE` | Document which port the app listens on | `EXPOSE 80` |
| `VOLUME` | Create a mount point for external storage | `VOLUME ["/data"]` |

---

## CMD vs ENTRYPOINT

| | `CMD` | `ENTRYPOINT` |
|--|-------|-------------|
| Purpose | Default args/command | Fixed command |
| Can be overridden? | Yes (by passing args to `docker run`) | No (unless `--entrypoint` flag) |
| Use when | You want a default that can change | You want a fixed executable |

---

## Sample Dockerfile

```dockerfile
# Start from official Python image
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy files
COPY requirements.txt .
COPY app.py .

# Install dependencies
RUN pip install -r requirements.txt

# Expose port
EXPOSE 5000

# Set environment variable
ENV APP_ENV=production

# Command to run the app
CMD ["python3", "app.py"]
```

---

## docker build Process

```bash
# Build image from current directory
docker build -t myapp:v1 .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with no cache (fresh build)
docker build --no-cache -t myapp:v1 .
```

**What happens during build:**
1. Docker reads each instruction top to bottom
2. Each instruction creates a **new layer**
3. Unchanged layers are **pulled from cache**
4. Final image is tagged and stored locally

---

## Image Tagging & Versioning

```bash
# Tag during build
docker build -t myapp:v1 .
docker build -t myapp:latest .

# Tag an existing image
docker tag myapp:v1 bhoomiijain/myapp:v1

# Push to Docker Hub
docker push bhoomiijain/myapp:v1
```

---

## Inspecting Images

```bash
# View image layers (history)
docker history myapp:v1

# Detailed image metadata in JSON
docker inspect myapp:v1

# See image size, tags, ID
docker images
```

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: Syllabus + Class practice*
