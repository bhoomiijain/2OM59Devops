# Dockerfile Basics

Dockerfile = plain text file with step-by-step instructions to automatically build Docker image. Docker reads top to bottom. Each instruction = one layer.

Build context = folder passed to docker build. All contents sent to daemon.

.dockerignore = exclude files from build context (like .gitignore)
Example:
node_modules/
*.log
.env
__pycache__/
.git

Core Dockerfile Instructions:
- FROM: base image. Example: FROM nginx:alpine
- RUN: run command during build. Example: RUN npm install
- COPY: copy files from host to image. Example: COPY app.js /app/
- ADD: like COPY + supports URLs & tar. ADD archive.tar.gz /app/
- CMD: default command when container starts (overridable). CMD ["node", "app.js"]
- ENTRYPOINT: fixed executable (not overridable). ENTRYPOINT ["nginx"]
- WORKDIR: set working directory. WORKDIR /app
- ENV: set environment variable. ENV PORT=3000
- EXPOSE: document port app uses. EXPOSE 80
- VOLUME: create mount point. VOLUME ["/data"]

CMD vs ENTRYPOINT:
- CMD: can be overridden by docker run. Use for default command
- ENTRYPOINT: cannot override. Use for fixed executable

Image Layering:
Each instruction = one layer. Layers cached - Docker skips rebuilding unchanged.

Instruction        Layer
FROM nginx:alpine → Layer 1 (base OS)
COPY index.html  → Layer 2
COPY default.conf → Layer 3
EXPOSE 80         → Layer 4 (metadata)

Best practice: put instructions that change least often at top so cache reused on rebuilds.

Project 1: Custom Nginx App

Files:
- index.html: HTML page
- default.conf: Nginx config
- Dockerfile: build instructions

index.html:
<!DOCTYPE html>
<html>
<head>
    <title>Docker App</title>
</head>
<body>
    <h1>Welcome to My Docker Nginx App</h1>
</body>
</html>

default.conf:
server {
    listen 80;
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}

Dockerfile:
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
COPY default.conf /etc/nginx/conf.d/default.conf
EXPOSE 80

Build & Run:
mkdir nginx-app
cd nginx-app
(create 3 files)
docker build -t custom-nginx:v1 .
docker run -d -p 8080:80 --name nginx-container custom-nginx:v1

Project 2: Node.js Express App

Files:
- app.js: Node code
- package.json: dependencies
- Dockerfile: build instructions

app.js:
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Docker Node App Running!");
});

app.listen(3000, "0.0.0.0", () => {
  console.log("Server running on port 3000");
});

package.json:
{
  "name": "node-docker-app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.2"
  }
}

Dockerfile:
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]

Build & Run:
mkdir node-app
cd node-app
(create 3 files)
docker build -t node-demo:v1 .
docker run -d -p 3000:3000 --name node-container node-demo:v1
docker ps
Open browser: http://localhost:3000

docker build commands:
docker build -t myapp:v1 . (. = current folder)
docker build -f Dockerfile.prod -t myapp:prod . (specific Dockerfile)
docker build --no-cache -t myapp:v1 . (ignore cache, rebuild)
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

