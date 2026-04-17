# Registries (INT332)

**Note**: File already contains comprehensive registry concepts. Structure is maintained.

Registry = storage for images. Push image → others can pull.

Main registries:
- Docker Hub (default)
- GitHub Container Registry (GHCR)
- AWS ECR
- Azure ACR
- Private registries (company)

Docker Hub:
https://hub.docker.com
Free + official images (nginx, ubuntu, mysql...)

Push Your Image:

1. Login:
docker login

2. Tag image:
docker tag myapp:v1 username/myapp:v1

3. Push:
docker push username/myapp:v1

4. Others can pull:
docker pull username/myapp:v1

Logout:
docker logout

GitHub Container Registry:
Store images with GitHub code. URL: ghcr.io

Need: GitHub Personal Access Token

Login: docker login ghcr.io

Tag: ghcr.io/yourusername/myapp:v1

Push:
docker push ghcr.io/yourusername/myapp:v1

Pull:
docker pull ghcr.io/yourusername/myapp:v1

Private Registries:
Used in enterprise for proprietary applications.

Registry | Provider | Use
ECR | AWS | AWS-hosted private
ACR | Azure | Azure-hosted private
GHCR | GitHub | GitHub-integrated
On-premise | Self-hosted | Full control

AWS ECR (Elastic Container Registry):

Login:
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account_id>.dkr.ecr.us-east-1.amazonaws.com

Tag and push:
docker tag myapp:v1 <account_id>.dkr.ecr.us-east-1.amazonaws.com/myapp:v1
docker push <account_id>.dkr.ecr.us-east-1.amazonaws.com/myapp:v1

Authentication & Access Tokens:

Why tokens instead of passwords?
- More secure
- Can be scoped (limited permissions)
- Can be revoked without changing password
- Required for CI/CD

Docker Hub Access Token:
Docker Hub → Account Settings → Security → New Access Token

Login using token:
docker login -u yourusername --password-stdin
paste token when prompted

CI/CD (GitHub Actions):
- name: Login to Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

Registry in CI/CD Pipeline:
Code pushed → CI builds image → Image pushed to Registry → CD pulls → Deploy

Registry is bridge between CI and CD

---

## Image Naming for Each Registry

Docker Hub: username/image:tag
Example: bhoomiijain/webapp:v1

GHCR: ghcr.io/username/image:tag
Example: ghcr.io/bhoomiijain/webapp:v1

AWS ECR: <account>.dkr.ecr.<region>.amazonaws.com/image:tag


