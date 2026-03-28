# Dockerfile

## What is it?

Recipe for making Docker images. Text file with instructions.

Docker reads line by line → creates image layers

## Basic Instructions

**FROM** - start image (base)
```
FROM ubuntu:22.04
```

**WORKDIR** - folder inside container
```
WORKDIR /app
```

**COPY** - copy files from computer to container
```
COPY app.py /app/
```

**RUN** - execute command during build
```
RUN pip install -r requirements.txt
```

**ENV** - environment variables
```
ENV PORT=8080
```

**EXPOSE** - document the port (doesn't actually expose)
```
EXPOSE 5000
```

**CMD** - default command when container starts
```
CMD ["python3", "app.py"]
```

## Example Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
COPY app.py .

RUN pip install -r requirements.txt

ENV APP_ENV=production

EXPOSE 5000

CMD ["python3", "app.py"]
```

## Build

```
docker build -t myapp:v1 .
```

- `-t` = tag/name
- `.` = looks in current folder

---
*Need to actually write a Dockerfile to understand this better*

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

