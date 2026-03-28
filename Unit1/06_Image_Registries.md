# Image Registries (Docker Hub etc)

## What = Registry?

Storage for images. Like GitHub but for containers.

**Registries:**
- Docker Hub (main one, free)
- AWS ECR
- Google Container Registry
- Azure ACR
- Private registries (company)

---

## Public vs Private

**Public:** Anyone can pull (no password)
- Free
- Base images (python, ubuntu, mysql...)
- Open source stuff
- Example: Docker Hub

**Private:** Need auth
- Company images
- Proprietary code
- Control who sees what
- AWS ECR, Azure ACR

---

## Docker Hub

Official registry

Format: `username/image:tag`

Example: `bhoomiijain/myapp:v1`

**Tags:**
- `latest` = newest
- `v1`, `v2` = versions
- `prod`, `dev`, `qa` = environments

## Push & Pull

**Push:** upload to registry
```
docker push bhoomiijain/myapp:v1
```

**Pull:** download from registry
```
docker pull python:3.11
```

Only new/changed layers = efficient

---
*Practice: Create Docker Hub account and push an image*

---


