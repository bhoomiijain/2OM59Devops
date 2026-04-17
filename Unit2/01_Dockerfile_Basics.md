# Docker Commands & Dockerfile Basics

Basic Docker Commands:

docker --version
Shows installed Docker version.
Example: Docker version 24.0.2, build abcdef123

docker info
Detailed Docker daemon information:
- Total containers running
- Total images
- Storage driver (overlay2, aufs)
- Docker root directory
- Server version
- Kernel version
- Operating system

docker -h
Shows help for all command

docker <command> --help
Shows help for specific command.
Example: docker run --help

Downloading and Managing Images:

docker pull image:tag
Download image from Docker Hub (or specified registry).
Example:
docker pull ubuntu:22.04 (pulls Ubuntu 22.04)
docker pull nginx (pulls latest Nginx)
docker pull python:3.11-slim (pulls Python 3.11 slim variant)
docker pull ghcr.io/bhoomiijain/myapp:v1 (pulls from GHCR)

docker images
List all local images.
Shows: repository, tag, image ID, size, creation date.

docker history image:tag
Show layers of an image - each instruction in Dockerfile creates one layer.
Example: docker history nginx:alpine
Layer info: created by (instruction), size, created date.

Useful: understand image composition and layer sizes.

docker inspect image:tag
Get detailed metadata about image (JSON format):
- Architecture
- OS
- Environment variables
- Exposed ports
- Volumes
- Layers/history

Running Containers:

docker run [OPTIONS] IMAGE [COMMAND]
Create and start a container from image.

Core run OPTIONS:

-it
Interactive terminal.
-i = keep STDIN open even without attach
-t = allocate pseudo-TTY (terminal)
Used together: get interactive shell inside container.

Example: docker run -it ubuntu bash (get shell inside Ubuntu)

-d
Run in background (daemon mode). Returns container ID.
Example: docker run -d nginx (start Nginx in background)

--name <name>
Give container a custom name (instead of random auto-generated).
Example: docker run -d --name web-server nginx

--rm
Auto-remove container when stops. Used for temp containers.
Example: docker run --rm ubuntu echo "Hello" (container removed after echo)

-p <host_port>:<container_port>
Port mapping. Make container port accessible from host.
Example: 
docker run -d -p 8080:80 nginx (host:8080 → container:80)
docker run -d -p 3000:3000 node-app
docker run -d -p 5432:5432 postgres

Multiple ports:
docker run -d -p 8080:80 -p 443:443 nginx

-e <KEY>=<VALUE>
Set environment variable inside container.
Example:
docker run -e MY_VAR=hello ubuntu echo $MY_VAR
docker run -d -e MYSQL_ROOT_PASSWORD=root123 mysql:8
docker run -d -e DB_HOST=localhost -e DB_USER=admin mongodb

Multiple variables:
docker run -d -e VAR1=val1 -e VAR2=val2 -e VAR3=val3 nginx

-v <volume>:<path>
Mount volume (persistent storage) into container.
Example:
docker run -d -v myvolume:/app/data nginx
docker run -d -v mysql_data:/var/lib/mysql mysql:8

Bind mount (host path → container path):
docker run -d -v $(pwd)/html:/usr/share/nginx/html nginx (Linux/Mac)
docker run -d -v C:\app\data:/data ubuntu (Windows)

Running Common Images:

Nginx (Web Server):
docker run -d -p 8080:80 --name webserver nginx
Access: http://localhost:8080

Ubuntu (Operating System):
docker run -it ubuntu bash
Get interactive shell inside Ubuntu.

MongoDB (Database):
docker run -d -p 27017:27017 --name mongo_db -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password123 mongo
Connect on localhost:27017

MySQL (Relational Database):
docker run -d -p 3306:3306 --name mysql_db -e MYSQL_ROOT_PASSWORD=root123 -e MYSQL_DATABASE=college mysql:8
Connect on localhost:3306

PostgreSQL (Relational Database):
docker run -d -p 5432:5432 --name postgres_db -e POSTGRES_PASSWORD=postgres123 postgres:15
Connect on localhost:5432

Node.js (Runtime):
docker run -d -p 3000:3000 --name node-app node:18
Or with custom code:
docker run -d -p 3000:3000 -v $(pwd):/app node:18 node /app/server.js

Python (Runtime):
docker run -d --name python-app python:3.11 python /app/script.py

Redis (In-Memory Database):
docker run -d -p 6379:6379 --name redis_db redis

HTTP Example (Testing):
docker run -it ubuntu bash
Inside: apt-get update && apt-get install -y curl
curl https://example.com

Echo Command (Testing):
docker run ubuntu echo "Hello World"
Output: Hello World

docker run -e MY_NAME=Bhoomi ubuntu echo "My name is $MY_NAME"
Output: My name is Bhoomi

Full run example:
docker run -it \
  --name dev-app \
  -p 8080:8080 \
  -e APP_ENV=development \
  -e DEBUG=true \
  -v $(pwd)/app:/app:rw \
  --rm \
  node:18 bash

Container Status and Inspection:

docker ps
List running containers.
Shows: container ID, image, command, creation time, status, ports, names.

docker ps -a
List ALL containers (running and stopped).

docker exec -it <container_id_or_name> bash
Execute command/open shell inside running container.
Example:
docker exec -it webserver bash (get shell inside nginx)
docker exec -it mysql_db mysql -u root -p (login to MySQL)

docker exec -it nginx_container <command>
Run specific command inside:
docker exec -it nginx_container ls /app
docker exec -it nginx_container ps aux

docker inspect <container_id_or_name>
Get detailed container metadata (JSON):
- IP address
- Port mappings
- Environment variables
- Mounted volumes
- Network configuration
- Running processes
- Resource limits

docker logs <container_id>
View container output/logs.

docker logs -f <container_id>
Follow logs in real-time (like tail -f).

docker stats <container_id>
See live CPU and memory usage.

docker top <container_id>
List processes running inside container.

Stopping and Removing Containers:

docker stop <container_id_or_name>
Gracefully stop running container. Process gets time to cleanup.
Example: docker stop webserver

docker start <container_id_or_name>
Restart stopped container.
Example: docker start webserver

docker restart <container_id_or_name>
Restart running container (stop + start).

docker kill <container_id_or_name>
Force stop immediately (no cleanup time).
Example: docker kill webserver

docker rm <container_id_or_name>
Delete stopped container (cannot delete running container without -f).
Example: docker rm webserver

docker rm -f <container_id_or_name>
Force delete (even if running).

docker container prune
Remove all stopped containers.
Cleanup unused containers safely.

Container Interaction with Host:

docker cp <container_id>:<path> <host_path>
Copy files FROM container TO host.
Example: docker cp webserver:/app/log.txt ~/Desktop/

docker cp <host_path> <container_id>:<path>
Copy files FROM host TO container.
Example: docker cp ~/Desktop/config.json webserver:/app/

Mount volume to share:
docker run -d -v ~/shared:/app/shared nginx
Files in ~/shared accessible inside /app/shared (both sync).

Execute commands from host into container:
docker exec -it webserver apt-get update
docker exec -it webserver npm install

Managing Images:

docker rmi <image_id_or_name>
Remove image locally.
Example: docker rmi nginx:latest

docker rmi -f <image_id_or_name>
Force remove (even if containers using it).

docker tag <image> <new_name>
Create new tag for image.
Example: docker tag nginx:latest myrepo/nginx:v1

Listing and Inspecting Images:

docker images
List all local images with size.

docker images --filter <filter>
Filter images by criteria.
Example: docker images --filter dangling=true (find unused images)

docker image prune
Remove unused images.

Image Creation, Building, and Pushing:

docker build -t <image_name>:<tag> <path>
Build image from Dockerfile.
Example: docker build -t myapp:v1 . (builds from ./Dockerfile)
docker build -t myapp:v1 -f Dockerfile.prod . (specific Dockerfile)

Build output: shows each layer build, final image ID and name.

docker build -t myapp:v1 --no-cache .
Force rebuild without using cached layers.

docker build --build-arg ARG_NAME=value -t myapp:v1 .
Pass build argument.

Pushing to Registry:

1. Tag image with registry path:
docker tag myapp:v1 username/myapp:v1

2. Login to registry:
docker login (Docker Hub)
docker login ghcr.io (GitHub)

3. Push:
docker push username/myapp:v1

4. Verify on registry:
Check Docker Hub or GHCR for image.

5. Others can pull:
docker pull username/myapp:v1

Build, Run, and Verify Workflow:

Create Dockerfile:
FROM nginx:alpine
COPY website/ /usr/share/nginx/html/
EXPOSE 80

Build image:
docker build -t mywebsite:v1 .

Run container:
docker run -d -p 8080:80 --name mysite mywebsite:v1

Verify running:
docker ps
curl http://localhost:8080

Push to Docker Hub:
docker tag mywebsite:v1 username/mywebsite:v1
docker login
docker push username/mywebsite:v1

Verify on hub:
docker pull username/mywebsite:v1
docker run -p 9090:80 username/mywebsite:v1

Dockerfile Basics:

Dockerfile = plain text file with step-by-step instructions to build Docker image automatically.
Docker reads top to bottom. Each instruction creates one layer.

Why Dockerfile?
- Automate image building process
- Version control (track changes)
- Reproducibility (same build always)
- Distribute images easily
- Scale deployments

Core Dockerfile Instructions:

FROM base_image:tag (REQUIRED - first instruction)
Specifies base image to build upon.

Examples: FROM ubuntu:22.04 | FROM python:3.11-slim | FROM node:18-alpine | FROM nginx:alpine

RUN command
Execute command during build time (not runtime).
Most common instruction.

Examples:
RUN apt-get update && apt-get install -y curl
RUN npm install
RUN pip install flask

COPY source destination
Copy files from host into image during build.

Examples:
COPY app.js /app/
COPY package.json /app/
COPY . /app (copy entire directory)

WORKDIR /path
Set default working directory for RUN, COPY, CMD, ENTRYPOINT.
If doesn't exist, Docker creates it.

Example:
WORKDIR /app
COPY package.json .  (copies to /app/package.json)
RUN npm install  (runs from /app directory)

ENV KEY value
Set environment variable available in container.

Examples:
ENV NODE_ENV production
ENV DATABASE_URL localhost:5432

EXPOSE port
Document which port application listens on.
Use with -p when running container to map port.

Examples: EXPOSE 80 | EXPOSE 3000 | EXPOSE 5432

CMD ["executable", "param1", "param2"]
Default command when container starts.
Can be overridden by docker run command.

Examples:
CMD ["npm", "start"]
CMD ["python", "app.py"]
CMD ["nginx", "-g", "daemon off;"]

ENTRYPOINT ["executable", "param"]
Fixed executable (cannot override with docker run).
Use when want fixed entry point.

Example: ENTRYPOINT ["node", "app.js"]

Building Images:

docker build -t image:tag .
Build image from Dockerfile in current directory.

docker build -t myapp:v1 . (current dir)
docker build -t myapp:v1 -f /path/to/Dockerfile . (specific path)
docker build -t myapp:v1 --no-cache . (rebuild without cache)

Image Creation Example 1: Nginx Web Server

Project Structure:
nginx-app/
├── Dockerfile
├── index.html
└── default.conf

Dockerfile:
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
COPY default.conf /etc/nginx/conf.d/default.conf
EXPOSE 80

Build & Run:
docker build -t custom-nginx:v1 .
docker run -d --name my-nginx -p 8080:80 custom-nginx:v1
Access: http://localhost:8080

Image Creation Example 2: Node.js App

Dockerfile:
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]

Build & Run:
docker build -t node-demo:v1 .
docker run -d --name my-node -p 3000:3000 node-demo:v1

Image Creation Example 3: Python Flask

Dockerfile:
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]

Build & Run:
docker build -t python-app:v1 .
docker run -d --name my-python -p 5000:5000 python-app:v1

Pushing Images to Registry:

docker tag image:old image:new
Create new tag for image.

docker login registry_url
Authenticate with registry (Docker Hub, GHCR, etc).

docker push registry/username/image:tag
Upload image to registry.

docker pull registry/username/image:tag
Download image from registry.

Pushing Workflow:

1. Build locally:
docker build -t myapp:v1 .

2. Tag for registry:
docker tag myapp:v1 ghcr.io/bhoomiijain/myapp:v1

3. Login:
docker login ghcr.io

4. Push:
docker push ghcr.io/bhoomiijain/myapp:v1

5. Others pull and use:
docker pull ghcr.io/bhoomiijain/myapp:v1
docker run ghcr.io/bhoomiijain/myapp:v1

Verifying Image:

docker images
List all local images showing size, ID, creation date.

docker history image:tag
Show all layers in image (each instruction creates layer).

docker inspect image:tag
Get detailed metadata (JSON):
- Architecture
- OS
- Exposed ports
- Environment variables
- Volumes
- Default command

Best Practices:

1. Use specific base image versions (not latest)
Bad: FROM ubuntu
Good: FROM ubuntu:22.04

2. Keep images small
Use alpine/slim variants
Remove unnecessary files

3. Minimize layers
Combine RUN commands: RUN apt-get update && apt-get install -y curl

4. Use .dockerignore
Exclude unnecessary files from build context

5. Run as non-root user
RUN useradd -m appuser
USER appuser

Common Base Images:

ubuntu:22.04 (general purpose)
alpine:latest (very small, ~5MB)
python:3.11 (Python pre-installed)
node:18 (Node.js pre-installed)
nginx:alpine (Nginx web server)
mysql:8 (MySQL database)
postgres:15 (PostgreSQL)
redis:7 (Redis cache)
golang:1.21 (Go runtime)

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

