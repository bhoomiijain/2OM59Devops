# 5. Registries — Docker Hub, GHCR & Private Registries

## Quick Recap: What is a Registry?

A registry is where Docker **images are stored and distributed** from.

```
Developer → [docker build] → [docker push] → Registry
Server    ← [docker pull] ←                 Registry
```

---

## Docker Hub

The **default and most popular** public registry.

**Website:** https://hub.docker.com

**Features:**
- Free public repositories
- Private repos (limited on free plan)
- Official images: `nginx`, `ubuntu`, `mysql`, `python`, `httpd`
- Automated builds from GitHub
- Image versioning & tagging

### Login and Push to Docker Hub

```bash
# Login
docker login
# Enter: Docker Hub username + password

# Tag your local image for Docker Hub
docker tag myapp:v1 yourusername/myapp:v1

# Push to Docker Hub
docker push yourusername/myapp:v1

# Pull from Docker Hub
docker pull yourusername/myapp:v1

# Logout
docker logout
```

---

## GitHub Container Registry (GHCR)

Stores Docker images **alongside your GitHub source code**.

**Registry URL:** `ghcr.io`

### Login and Push to GHCR

```bash
# Generate a GitHub Personal Access Token (PAT)
# → GitHub → Settings → Developer Settings → Personal Access Tokens
# Required scopes: read:packages, write:packages, delete:packages

# Login to GHCR
echo YOUR_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin

# Tag image for GHCR
docker tag myapp:v1 ghcr.io/yourusername/myapp:v1

# Push
docker push ghcr.io/yourusername/myapp:v1

# Pull
docker pull ghcr.io/yourusername/myapp:v1
```

---

## Private Registries

Used in enterprise environments for **proprietary applications**.

| Registry | Provider | Use |
|----------|----------|-----|
| ECR | AWS | AWS-hosted private registry |
| ACR | Azure | Azure-hosted private registry |
| GHCR | GitHub | GitHub-integrated registry |
| On-premise | Self-hosted | Full control, internal network |

### AWS ECR (Elastic Container Registry)

```bash
# Login to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  <account_id>.dkr.ecr.us-east-1.amazonaws.com

# Tag and push
docker tag myapp:v1 <account_id>.dkr.ecr.us-east-1.amazonaws.com/myapp:v1
docker push <account_id>.dkr.ecr.us-east-1.amazonaws.com/myapp:v1
```

---

## Authentication & Access Tokens

### Why tokens instead of passwords?
- More secure
- Can be **scoped** (limited permissions)
- Can be **revoked** without changing password
- Required for CI/CD pipelines

### Docker Hub Access Token

```
Docker Hub → Account Settings → Security → New Access Token
```

```bash
# Login using token
docker login -u yourusername --password-stdin
# paste token when prompted
```

### Using Tokens in CI/CD (GitHub Actions example)

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

---

## Registry in CI/CD Pipeline

```
Code pushed to GitHub
        ↓
CI builds Docker image
        ↓
Image pushed to Registry (Docker Hub / GHCR / ECR)
        ↓
CD pulls image from Registry
        ↓
Container deployed to server/cloud
```

> The registry is the **bridge between CI and CD**.

---

## Image Naming for Each Registry

| Registry | Image Name Format | Example |
|----------|------------------|---------|
| Docker Hub | `username/image:tag` | `bhoomiijain/webapp:v1` |
| GHCR | `ghcr.io/username/image:tag` | `ghcr.io/bhoomiijain/webapp:v1` |
| AWS ECR | `<account>.dkr.ecr.<region>.amazonaws.com/image:tag` | — |

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: Slides + Handwritten notes (Pages 7–9)*
