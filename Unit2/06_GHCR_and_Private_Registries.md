# 6. GitHub Container Registry (GHCR) — Step-by-Step Guide

> **GHCR** lets you store Docker images **directly alongside your GitHub source code**.  
> Registry URL: `ghcr.io`

---

## What You'll Learn
- Build a Docker image
- Push image to GHCR
- Pull image from GHCR
- Understand authentication with PAT tokens

---

## Step 1 — Create a GitHub Personal Access Token (PAT)

A **PAT (Personal Access Token)** acts as your password for Docker when pushing to GHCR.

> ⚠️ Use your PAT, NOT your GitHub password, when logging into GHCR.

**How to generate:**

1. Go to: **GitHub → Settings → Developer Settings → Personal Access Tokens**  
   Direct link: https://github.com/settings/tokens

2. Choose: **Classic Token**

3. Enable these permissions:
   - ✅ `write:packages`
   - ✅ `read:packages`
   - ✅ `delete:packages` *(optional)*

4. Click **Generate Token**

5. **Copy it immediately** — you will NOT see it again

---

## Step 2 — Create a Simple Demo Project

```bash
# Create project folder
mkdir ghcr-demo
cd ghcr-demo
```

**Create `index.html`:**
```html
<h1>GHCR Demo Success!</h1>
```

**Create `Dockerfile`:**
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

---

## Step 3 — Build the Docker Image

```bash
# Replace "yourusername" with your actual GitHub username
docker build -t ghcr.io/yourusername/ghcr-demo .

# Example:
docker build -t ghcr.io/bhoomiijain/ghcr-demo .
```

> **Image naming for GHCR:** `ghcr.io/<github-username>/<image-name>`

---

## Step 4 — Login to GHCR

```bash
docker login ghcr.io
```

When prompted:
- **Username** → your GitHub username
- **Password** → your PAT token (NOT your GitHub password)

**Alternative (non-interactive, good for CI/CD):**
```bash
echo YOUR_PAT_TOKEN | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

---

## Step 5 — Push Image to GHCR

```bash
docker push ghcr.io/yourusername/ghcr-demo

# Example:
docker push ghcr.io/bhoomiijain/ghcr-demo
```

**What happens:**
- Image layers upload from local machine → GitHub registry
- Only **new/changed layers** are pushed (efficient)

---

## Step 6 — Verify on GitHub

1. Go to your GitHub profile
2. Click **Packages** tab
3. You will see:
   - ✅ Image name
   - ✅ Visibility (public/private)
   - ✅ Version/tag

---

## Step 7 — Pull Image Back (Proof of Success)

```bash
# Delete local image first
docker rmi ghcr.io/yourusername/ghcr-demo

# Pull from GHCR
docker pull ghcr.io/yourusername/ghcr-demo
```

---

## Step 8 — Run Container (Final Visual Proof)

```bash
docker run -p 8080:80 ghcr.io/yourusername/ghcr-demo

# Open browser → http://localhost:8080
# Expected: "GHCR Demo Success!"
```

---

## GHCR vs Docker Hub

| Feature | Docker Hub | GHCR |
|---------|-----------|------|
| Provider | Docker Inc. | GitHub |
| Free private repos | Limited (1) | Unlimited |
| Integration | General purpose | Native GitHub/Actions |
| Auth method | Docker Hub token | GitHub PAT |
| Image URL format | `username/image:tag` | `ghcr.io/username/image:tag` |
| Best for | Public open-source images | Projects hosted on GitHub |

---

## Authentication & Access Tokens — Summary

### Why Tokens Instead of Passwords?

| Reason | Explanation |
|--------|-------------|
| **Scoped** | Token only has the permissions you choose |
| **Revocable** | Revoke without changing your password |
| **Auditable** | You can see when/where it was used |
| **Required for CI/CD** | Pipelines can't use 2FA passwords |

### Docker Hub Access Token

```
Docker Hub → Account Settings → Security → New Access Token
```

```bash
docker login -u yourusername
# Paste token when prompted (not your password)
```

### Using Tokens in GitHub Actions (CI/CD)

```yaml
- name: Login to GHCR
  uses: docker/login-action@v2
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

---

## Private Registry — Run Your Own

You can also self-host a Docker registry:

```bash
# Run a private registry locally on port 5000
docker run -d -p 5000:5000 --name registry registry:2

# Tag your image for local registry
docker tag nginx localhost:5000/myimage

# Push to local registry
docker push localhost:5000/myimage

# Pull from local registry
docker pull localhost:5000/myimage
```

---

## Image Naming Convention — All Registries

| Registry | Format | Example |
|----------|--------|---------|
| Docker Hub | `username/image:tag` | `bhoomiijain/webapp:v1` |
| GHCR | `ghcr.io/username/image:tag` | `ghcr.io/bhoomiijain/webapp:v1` |
| AWS ECR | `<account>.dkr.ecr.<region>.amazonaws.com/image:tag` | — |
| Local | `localhost:5000/image:tag` | `localhost:5000/myapp:v1` |

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: GHCR.pdf + New PPT slides 70–73*
