# Image Registries

Registry = storage for images. Like GitHub but for containers.

Registries: Docker Hub (main one, free), AWS ECR, Google Container Registry, Azure ACR, private registries (company)

Public registries: anyone can pull (no password). Free. Has base images (python, ubuntu, mysql), open source. Example: Docker Hub.

Private registries: need auth. For company images, proprietary code, control access. AWS ECR, Azure ACR, self-hosted.

Docker Hub = official registry. Format: username/image:tag. Example: bhoomiijain/myapp:v1

Tags:
- latest = newest
- v1, v2 = versions
- prod, dev, qa = environments

Push = upload to registry
docker push bhoomiijain/myapp:v1

Pull = download from registry
docker pull python:3.11

Only new/changed layers transfer = efficient


