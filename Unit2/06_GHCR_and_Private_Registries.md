# GitHub Container Registry (GHCR)

GHCR lets you store Docker images directly alongside GitHub source code. Registry URL: ghcr.io

What You'll Learn:
- Build Docker image
- Push image to GHCR
- Pull image from GHCR
- Understand authentication with PAT tokens

Step 1: Create GitHub Personal Access Token (PAT)

PAT acts as password for Docker when pushing to GHCR.

Warning: Use your PAT, NOT GitHub password, when logging into GHCR.

How to generate:
1. Go to: GitHub → Settings → Developer Settings → Personal Access Tokens
   Direct: https://github.com/settings/tokens
2. Choose: Classic Token
3. Enable: write:packages, read:packages, delete:packages (optional)
4. Click: Generate Token
5. Copy immediately - won't see again

Step 2: Create Simple Demo Project

mkdir ghcr-demo
cd ghcr-demo

Create index.html:
<h1>GHCR Demo Success!</h1>

Create Dockerfile:
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html

Step 3: Build Docker Image

Replace "yourusername" with actual GitHub username:
docker build -t ghcr.io/yourusername/ghcr-demo .

Example:
docker build -t ghcr.io/bhoomiijain/ghcr-demo .

Image naming for GHCR: ghcr.io/<github-username>/<image-name>

Step 4: Login to GHCR

docker login ghcr.io

Prompted:
- Username: your GitHub username
- Password: your PAT token (NOT GitHub password)

Alternative (non-interactive):
echo YOUR_PAT_TOKEN | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin

Step 5: Push Image to GHCR

docker push ghcr.io/yourusername/ghcr-demo

Example:
docker push ghcr.io/bhoomiijain/ghcr-demo

What happens: Image layers upload from local to GitHub registry. Only new/changed layers uploaded (efficient).

Step 6: Verify on GitHub

1. Go GitHub profile
2. Click Packages tab
3. You see: image name, visibility (public/private), version/tag

Step 7: Pull Image Back (Proof)

Delete local first:
docker rmi ghcr.io/yourusername/ghcr-demo

Pull from GHCR:
docker pull ghcr.io/yourusername/ghcr-demo

Step 8: Run Container (Final Proof)

docker run -p 8080:80 ghcr.io/yourusername/ghcr-demo

Open browser: http://localhost:8080
Expected: GHCR Demo Success!

---

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
