# Container Images & Layers

What is an Image?

Image = blueprint/template for containers. Read-only. Immutable once built.

Contains:
- Base OS (lightweight Linux: Alpine, Ubuntu slim)
- Runtime (Python, Java, Node.js)
- Dependencies (libraries, packages)
- Application code
- Configuration

Example: python:3.11-slim image contains Linux + Python 3.11 + pip pre-installed.

When you run container: Docker makes container from image and adds writable top layer (ephemeral).

Image vs Container:

Image = template (read-only, stored on disk, reusable)
Container = running instance (writable, has ephemeral storage, temporary)

Multiple containers can run from same image.
Each container gets own writable layer.
Modifications inside container isolated to that container.

Image Naming Convention:

Full image name format: registry/namespace/image:tag

Example: docker.io/library/python:3.11
- registry: docker.io (Docker Hub, default if omitted)
- namespace: library (default namespace, free images)
- image: python
- tag: 3.11 (version identifier)

Another example: ghcr.io/bhoomiijain/myapp:v1.0
- registry: ghcr.io (GitHub Container Registry)
- namespace: bhoomiijain (username)
- image: myapp
- tag: v1.0

Short form (commonly used):
python:3.11
- Uses default registry (docker.io)
- Uses default namespace (library)
- Image: python
- Tag: 3.11

Tags explained:
- latest = newest version (default if omitted)
- v1, v2, v1.0.1 = version numbers
- prod, staging, dev = environment
- arm64, amd64 = architecture

Example tags:
myapp:latest (newest version)
myapp:v1.0 (specific version)
myapp:prod-v2 (production version 2)
myapp:1.0-arm64 (version 1.0 for ARM processors)

Registries in naming:
- docker.io = Docker Hub (default, no need to specify)
- ghcr.io = GitHub Container Registry
- quay.io = Quay.io registry
- gcr.io = Google Container Registry
- ecr.amazonaws.com = AWS Elastic Container Registry
- myregistry.azurecr.io = Azure Container Registry
- private-registry.company.com = Private company registry

Public vs Private namespace:
- Public: registry/library/image - anyone can pull
- Private: registry/username/image - only you or auth users can pull

Image Layers:

Docker images are built as stack of layers. Each layer = snapshot of filesystem at that stage.

Example layer structure (Python web app):
Layer 1 (Base): Ubuntu Linux OS (120MB)
Layer 2: Python 3.11 runtime (400MB)
Layer 3: pip packages (Flask, requests, etc) (50MB)
Layer 4: Application code (10KB)
Result: Image totals ~570MB

Dockerfile lines = layers:

FROM python:3.11-slim          (Layer 1: inherits Python image)
RUN pip install flask          (Layer 2: Flask library)
COPY app.py /app/              (Layer 3: application code)
WORKDIR /app                   (Layer 4: working directory setting)
CMD ["python", "app.py"]       (Layer 5: command to run)

Why layers important?

1. Immutability: Once layer created, never changes. Ensures reproducibility.
2. Caching: Docker reuses layers. If Layer 1 unchanged, Docker skips rebuilding it (faster builds).
3. Sharing: Same layer used across multiple images (saves disk space). Example: 100 Python apps share python:3.11 layer.
4. Efficiency: Only changed layers rebuilt/repulled. Saves bandwidth.
5. Storage: Total image size = sum of all layers. Sharing layers = smaller total disk footprint.

Container writable layer:

When container runs:
- Docker stacks all image layers (read-only)
- Docker adds top writeable layer (ephemeral)
- Container changes go to top layer
- Original image layers untouched

Example: Running 5 containers from python:3.11 image
- Shared image layers (read-only): 600MB used once
- Each container's writeable layer: 5-10MB each
- Total storage: ~600MB + (5×10MB) = ~650MB
- Without layer sharing: 5×600MB + 5×10MB = 3050MB
- Result: 80% storage savings!

Copy-on-Write (CoW):

Mechanism where changes create copies only when needed.

Example:
1. Image has file /etc/config (read-only)
2. Container modifies /etc/config
3. Docker copies file to container's writable layer
4. Container now has modified copy
5. Original file in image untouched
6. Other containers still see original file

Result: Efficient storage. Only changed files take extra space.

Image distribution:

Image stored as layers in registry.
When pushing: only new/changed layers pushed
When pulling: only new/changed layers pulled
Rest reused from local cache or shared layers

Example: Push myapp:v2 after v1
v1 has: Layer A, B, C, D
v2 has: Layer A, B, C, D, E (Layer E is new)
Push process: only Layer E uploaded (Layers A-D already on registry)
Result: Fast push/efficient bandwidth

Base images and inheritance:

FROM ubuntu:22.04 = base image (starts from here)
FROM python:3.11 = base image (includes ubuntu + python)
FROM node:18-alpine = base image (includes alpine + node)

Custom app layers built on top of base image layers.

Example image composition (production web app):

FROM node:18-alpine
RUN npm install -g pm2
COPY package.json /app/
RUN npm install
COPY src/ /app/src/
ENV NODE_ENV=production
EXPOSE 3000
CMD ["npm", "start"]

Layers created:
1. node:18-alpine (base: 150MB - Alpine Linux + Node.js)
2. pm2 installed (160MB total)
3. package.json copied (160MB)
4. dependencies installed (npm packages added) (180MB)
5. source code copied (190MB)
6. environment variables set (no size change, metadata)
7. port exposed (no size change, metadata)
8. startup command defined (no size change, metadata)

Final image: ~190MB containing all layers


