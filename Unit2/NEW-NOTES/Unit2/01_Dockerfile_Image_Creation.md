# 1. Dockerfile & Image Creation — Full Guide

## What is a Dockerfile?

A **Dockerfile** is a plain text file with step-by-step instructions to **automatically build a Docker image**.

> Docker reads instructions top to bottom — each instruction creates one image layer.

---

## Build Context & .dockerignore

**Build context** = the folder you pass to `docker build` (all its contents are sent to the Docker daemon).

**.dockerignore** file = tells Docker what to **exclude** from the build context (like `.gitignore`).

```
# .dockerignore example
node_modules/
*.log
.env
__pycache__/
.git
```

---

## Core Dockerfile Instructions

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM nginx:alpine` |
| `RUN` | Run command during build | `RUN npm install` |
| `COPY` | Copy files from host to image | `COPY app.js /app/` |
| `ADD` | Like COPY + supports URLs & tar extraction | `ADD archive.tar.gz /app/` |
| `CMD` | Default command when container starts (overridable) | `CMD ["node", "app.js"]` |
| `ENTRYPOINT` | Fixed executable (not overridable) | `ENTRYPOINT ["nginx"]` |
| `WORKDIR` | Set working directory inside container | `WORKDIR /app` |
| `ENV` | Set environment variable in image | `ENV PORT=3000` |
| `EXPOSE` | Document which port the app uses | `EXPOSE 80` |
| `VOLUME` | Create a mount point | `VOLUME ["/data"]` |

### CMD vs ENTRYPOINT

| | CMD | ENTRYPOINT |
|--|-----|-----------|
| Can be overridden by `docker run`? | ✅ Yes | ❌ No (use `--entrypoint` flag) |
| Use for | Default command/args | Fixed executable |

---

## Image Layering in Build

Each instruction = one layer. Layers are **cached** — Docker skips rebuilding unchanged layers.

```
Instruction          Layer
─────────────────────────────────
FROM nginx:alpine  → Layer 1 (base OS)
COPY index.html    → Layer 2
COPY default.conf  → Layer 3
EXPOSE 80          → Layer 4 (metadata only)
```

> **Best practice:** Put instructions that change least often at the top (e.g., `FROM`, `RUN apt install`) so cache is used for them on rebuilds.

---

## Project 1 — Custom Nginx App

### Files needed:

**index.html**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Medium Level Docker App</title>
</head>
<body>
    <h1>Welcome to My Custom Docker Nginx App</h1>
    <p>This is a medium-level Docker project.</p>
</body>
</html>
```

**default.conf**
```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

**Dockerfile**
```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
COPY default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
```

### Build & Run:
```bash
mkdir nginx-app
cd nginx-app
# (create the 3 files above)

# Build the image
docker build -t custom-nginx:v1 .

# Run the container
docker run -d -p 8080:80 --name nginx-container custom-nginx:v1

# Open browser → http://localhost:8080
```

---

## Project 2 — Node.js Express App

### Files needed:

**app.js**
```javascript
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Docker Node App Running!");
});

app.listen(3000, "0.0.0.0", () => {
  console.log("Server running on port 3000");
});
```

**package.json**
```json
{
  "name": "node-docker-app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

**Dockerfile**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]
```

### Build & Run:
```bash
mkdir node-app
cd node-app
# (create the 3 files above)

# Build
docker build -t node-demo:v1 .

# Run
docker run -d -p 3000:3000 --name node-container node-demo:v1

docker ps
# Open browser → http://localhost:3000
# Expected output: Docker Node App Running!
```

---

## docker build Process — Step by Step

```bash
# Basic build (. = current folder as build context)
docker build -t myapp:v1 .

# Build from specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Force rebuild ignoring cache
docker build --no-cache -t myapp:v1 .
```

**What happens:**
1. Docker reads Dockerfile line by line
2. Each instruction creates a layer
3. Unchanged layers → served from cache (fast)
4. Final image stored locally with the given tag

---

## Image Tagging & Versioning

```bash
# Tag during build
docker build -t myapp:v1 .
docker build -t myapp:latest .

# Tag existing image for Docker Hub
docker tag myapp:v1 yourusername/myapp:v1

# Tag for GHCR
docker tag myapp:v1 ghcr.io/yourusername/myapp:v1

# Push
docker push yourusername/myapp:v1
```

---

## Inspecting Images

```bash
# View all layers + commands that created them
docker history custom-nginx:v1

# Full metadata in JSON (size, layers, config, etc.)
docker inspect custom-nginx:v1

# List images with size/tag/ID
docker images

# Check storage driver
docker info | grep "Storage Driver"
```

---

## Windows-specific Notes (from class)

On Windows, use `notepad` to create files:
```powershell
notepad index.html
notepad Dockerfile
notepad default.conf

# List files to verify
dir

# If Dockerfile was saved as Dockerfile.txt, rename it:
Rename-Item Dockerfile.txt Dockerfile
```

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: New PPT slides 45–65 (Basic_Docker_commands + Unit_2)*
