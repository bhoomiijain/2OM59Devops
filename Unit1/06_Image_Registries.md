# 6. Image Registries & Distribution

## What is an Image Registry?

An **Image Registry** is a **centralized repository** used to store, manage, version, and distribute container images.

> Similar to:
> - **GitHub** → for source code
> - **Maven Central** → for Java libraries
> - **Docker Hub** → for container images

---

## Why Registries are Needed

| Without Registry | With Registry |
|-----------------|---------------|
| Each system must build images independently | Build once, deploy everywhere |
| High inconsistency across environments | Version-controlled environments |
| Manual software installation | Faster deployments |
| No version control of environments | Rollback capability |

---

## What a Registry Enables:
- Sharing images across teams and environments
- Consistent deployment in development, testing, and production
- Integration with CI/CD pipelines
- Registry acts as the **bridge between CI and CD**

---

## Types of Image Registries

### 1. Public Image Registries
**Open access** to images — anyone can pull.

**Characteristics:**
- Images are publicly visible
- Anyone can pull images
- Often free
- Used for base images and open-source apps

**Example:** Docker Hub (free public repos)

### 2. Private Image Registries
**Restricted access** using authentication & authorization.

**Characteristics:**
- Secure and controlled access
- Used in enterprises
- Stores proprietary applications
- Integrated with IAM and RBAC

**Examples:**
- AWS Elastic Container Registry (ECR)
- Azure Container Registry (ACR)
- GitHub Container Registry (GHCR)
- On-premise private registry

---

## Image Naming Convention

```
<registry>/<namespace>/<image>:<tag>
```

| Part | Meaning | Example |
|------|---------|---------|
| registry | Location of registry | `docker.io` |
| namespace | User/organization name | `bhoomiijain` |
| image | Name of the application | `webapp` |
| tag | Version or label | `v1`, `latest` |

**Full example:** `docker.io/bhoomiijain/webapp:v1`

---

## Tagging Strategies

| Tag | Use Case |
|-----|----------|
| `latest` | Default (not recommended for production) |
| `v1`, `v2` | Versioned releases |
| `dev` | Development build |
| `qa` | Testing build |
| `prod` | Stable production build |

---

## Image Distribution — Push & Pull

### Push Operation (Developer → Registry)
- Uploads image layers from local system to registry
- Requires authentication
- Only **new layers** are pushed (efficient)

### Pull Operation (Registry → Server)
- Downloads image layers from registry
- Uses **caching** to avoid re-downloading existing layers
- Faster on repeated deployments

```
Developer → [docker build] → [docker push] → Registry
Server    ← [docker pull] ← Registry
```

---

## Docker Hub — The Default Registry

Docker Hub is the **centralized repository for Docker images**.

**Features:**
- Public and private repositories
- Official images (nginx, ubuntu, mysql, etc.)
- Automated builds
- Important in DevOps pipelines

**DockerHub** also enables **global sharing & versioning** of application images (free).

---

> 📅 *Notes from Class — Unit I*  
> 📖 *Reference: Slides (Slides 39–51) + Handwritten notes (Pages 7–9)*
