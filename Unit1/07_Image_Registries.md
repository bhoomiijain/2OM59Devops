# Image Registries

What is a Registry?

Registry = centralized storage for Docker images. Like GitHub but for containers.

Purpose: Store, version, distribute container images.

Users push images to registry (upload).
Users pull images from registry (download).

Public vs Private Registries:

Public Registries:
- Anyone can pull images (no authentication needed)
- Free to use
- Contains base images (python, ubuntu, mysql, nodejs)
- Open source projects share images
- Example: Docker Hub

Private Registries:
- Authentication required (username/password or token)
- For company proprietary images
- For sensitive/internal applications
- Controlled access (only authorized users can pull)
- Examples: AWS ECR, Azure ACR, GitHub Container Registry, self-hosted registries

Popular Registries:

1. Docker Hub (docker.io)
Official registry. Largest collection of public images.
Format: username/image:tag
Example: bhoomiijain/myapp:v1
Free tier available. Unlimited public repos, limited private repos.

2. GitHub Container Registry (ghcr.io)
GitHub's registry for container images.
Format: ghcr.io/username/image:tag
Example: ghcr.io/bhoomiijain/myapp:v1
Integrated with GitHub repositories. Link image to code repo.

3. AWS Elastic Container Registry (ecr)
Format: 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:v1
Private by default. Pay per GB stored.

4. Azure Container Registry (acr)
Format: myregistry.azurecr.io/myapp:v1
Integrated with Azure services. Premium support.

5. Google Container Registry (gcr.io)
Format: gcr.io/project-id/myapp:v1
Integrated with Google Cloud. Pay per storage/bandwidth.

6. Quay.io Registry (quay.io)
Format: quay.io/username/myapp:v1
Alternative public/private registry. Team collaboration features.

7. Self-hosted Registry
Format: mycompany-registry.com/myapp:v1
Docker image registry you run yourself. Total control. Requires maintenance.

Image Naming in Registries:

Full format: registry/namespace/image:tag

Example: docker.io/library/python:3.11
- registry (required in full form): docker.io
- namespace (requires /): library
- image (required): python
- tag (optional, defaults to latest): 3.11

Short forms (defaults assumed):
- python:3.11 (uses docker.io/library/python:3.11)
- ubuntu:22.04 (uses docker.io/library/ubuntu:22.04)
- ghcr.io/bhoomiijain/myapp:v1 (full form, no defaults)

Tags:

Tag = version identifier. Helps identify specific image version.

Common tag patterns:
- latest (newest, default if not specified)
- v1, v2, v1.0.1 (semantic versioning)
- 3.11, 3.12 (runtime versions)
- prod, staging, dev (environments)
- 20250417 (date-based)
- arm64, amd64 (architecture)
- slim, alpine (variants)

Example:
python:latest (newest Python image)
python:3.11 (Python 3.11)
python:3.11-slim (Python 3.11, slim variant)
python:3.11-alpine (Python 3.11, Alpine Linux variant)
myapp:v1.0-prod (application version 1.0, production)
myapp:20250417-amd64 (built on this date, for AMD64 architecture)

Multiple tags for same image:
- docker tag myapp:v1.0 myapp:latest (v1.0 also tagged as latest)
- docker tag myapp:v1.0 myapp:prod (v1.0 also tagged as prod)
- Same image, multiple names, saves storage

Image Distribution Process:

Step 1: Build
Developer creates Dockerfile.
Run: docker build -t myregistry/myapp:v1 .
Docker creates layers, builds image locally.
Image stored in local Docker storage.

Step 2: Login to Registry
Before pushing, authenticate with registry.
docker login docker.io (login to Docker Hub)
docker login ghcr.io (login to GitHub registry)
Enter username and password (or token).
Credentials stored locally (in ~/.docker/config.json).

Step 3: Push
Upload image layers to registry.
docker push myregistry/myapp:v1

Push process:
1. Docker reads image layers
2. Connects to registry
3. Checks which layers already exist on registry
4. Uploads only new/changed layers
5. Updates image manifest (metadata)
6. Image now available for others to pull

Example push output:
docker push ghcr.io/bhoomiijain/myapp:v1

Pushing 3 layers:
layer-1: ████████████████ 100% Complete
layer-2: ████████████████ 100% Complete
layer-3: ████████████████ 100% Complete
Digest: sha256:abc123... (unique identifier)
Status: Pushed

Step 4: Other users pull
docker pull ghcr.io/bhoomiijain/myapp:v1

Pull process:
1. User's Docker daemon contacts registry
2. Registry returns image metadata (layers, sizes)
3. Docker daemon checks which layers already local
4. Downloads only new/missing layers
5. Assembles layers into complete image
6. Image ready to run

Example pull output:
docker pull python:3.11

Pulling from library/python:
e7c (status): Pull complete
f1e (status): Pull complete
.... (more layers)
Status: Downloaded newer image for python:3.11

Pull and Push Mechanics:

Pull:

docker pull ubuntu:22.04

Steps:
1. Parse image name: registry=docker.io, namespace=library, image=ubuntu, tag=22.04
2. Connect to docker.io
3. Authenticate (if private registry)
4. Request image metadata
5. Docker returns all layer digests (SHA256 hashes)
6. Docker client checks local cache (do I have these layers?)
7. For each missing layer: download from registry
8. Verify layer integrity (checksums)
9. Extract layer to Docker storage
10. Image ready to use (all layers present)

Layer reuse during pull:
Pull python:3.11 (first time): downloads all layers (400MB)
Pull python:3.11 again: uses local cache (instant)
Pull python:3.12 (shares ubuntu base layer): downloads only new layers (50MB), reuses 350MB

Push:

docker push myregistry/myapp:v1.0

Steps:
1. Parse image name: registry, namespace, image, tag
2. Connect to registry
3. Authenticate with credentials
4. Get image from local Docker storage
5. Read all layers
6. For each layer:
   a. Create layer digest (SHA256 hash of layer content)
   b. Check if layer already exists on registry (HEAD request)
   c. If exists: skip, layer already pushed
   d. If missing: upload compressed layer data
7. Create image manifest (lists all layers, pointers to registry locations)
8. Upload manifest
9. Push complete, image now on registry
10. Tag updated: registry knows image:v1.0 exists

Layer reuse during push:
Push myapp:v1 (first time): uploads all 5 layers (300MB)
Push myapp:v2 (only code changed): Layer 1-4 already on registry, uploads only Layer 5 (10MB)
Result: 97% faster push!

Registry interaction commands:

docker pull image:tag
Pull image from registry to local.

docker push registry/image:tag
Push local image to registry.

docker login registry.com
Authenticate with registry.

docker logout registry.com
Remove credentials.

docker search python
Search Docker Hub for image (only works on Docker Hub).

docker tag image:tag registry/image:tag
Retag image before push (necessary to push to registry).

docker rmi image:tag
Remove image locally.

docker images
List all local images.

Example workflow:

1. Pull base image:
docker pull python:3.11

2. Build custom image:
docker build -t myapp:v1 .

3. Tag for registry:
docker tag myapp:v1 ghcr.io/bhoomiijain/myapp:v1

4. Login to registry:
docker login ghcr.io

5. Push to registry:
docker push ghcr.io/bhoomiijain/myapp:v1

6. Other user pulls:
docker pull ghcr.io/bhoomiijain/myapp:v1

7. User runs container:
docker run ghcr.io/bhoomiijain/myapp:v1

Efficiency of layer sharing:

Three developers, all using Node.js apps:
Dev A builds: node:18-alpine + code A (total: 160MB layer-size)
Dev B builds: node:18-alpine + code B (layer-size: only 20MB new)
Dev C builds: node:18-alpine + code C (layer-size: only 15MB new)

Without layer sharing:
Total registry storage: 160MB + 160MB + 160MB = 480MB

With layer sharing:
Shared node:18-alpine layer: 160MB (once)
Dev A code: 20MB
Dev B code: 15MB
Dev C code: 10MB
Total registry storage: 160MB + 20MB + 15MB + 10MB = 205MB

Result: 57% storage savings! Efficiency increases with more users/services.


