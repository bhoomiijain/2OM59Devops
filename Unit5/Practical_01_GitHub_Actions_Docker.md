# Practical 1 — GitHub Actions Workflow with Docker

## Objective
Build a **Flask web app**, containerize it with Docker, and set up a **GitHub Actions CI pipeline** that automatically builds the Docker image every time code is pushed.

---

## Project Structure
```
my-docker-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── .github/
    └── workflows/
        └── ci-cd.yml
```

---

## Step 1 — Create Project Folder

```bash
mkdir my-docker-app
cd my-docker-app
```

---

## Step 2 — Create `app.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from Docker CI/CD!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

---

## Step 3 — Create `requirements.txt`

```
Flask
```

---

## Step 4 — Create `Dockerfile`

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

---

## Step 5 — Create GitHub Actions Workflow

Create the file at: `.github/workflows/ci-cd.yml`

```yaml
name: CI-CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Build Docker Image
        run: docker build -t my-docker-app .

      - name: List Docker Images
        run: docker images
```

**On Windows, create the folder structure:**
```powershell
mkdir .github
mkdir .github\workflows
# Create ci-cd.yml inside .github\workflows\
```

---

## Step 6 — Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/my-docker-app.git
git push -u origin main
```

**If you get proxy issues on Windows:**
```powershell
netsh winhttp reset proxy
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "", "User")
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "", "User")
git -c http.proxy= -c https.proxy= push -u origin main
```

**If workflow file isn't in the right place:**
```powershell
move ci-cd.yaml .github\workflows\ci-cd.yml
git add .
git commit -m "Fix workflow path"
git push
```

---

## Step 7 — Verify on GitHub

1. Go to your repo on GitHub
2. Click the **Actions** tab
3. You should see the `CI-CD Pipeline` workflow running
4. ✅ Green checkmark = success

---

## Step 8 — Add GitHub Secrets (for future push to Docker Hub)

Go to: `Settings → Secrets and variables → Actions → New repository secret`

Add:
- `DOCKERHUB_USERNAME` → your Docker Hub username
- `DOCKERHUB_TOKEN` → your Docker Hub access token

---

## Run App Locally (Important for Viva)

```bash
# Build Docker Image
docker build -t my-docker-app .

# Run Container
docker run -p 5000:5000 my-docker-app

# Open in Browser
# http://localhost:5000
# Expected output: Hello from Docker CI/CD!
```

---

## What This Workflow Does

```
Push code to GitHub (main branch)
        ↓
GitHub Actions triggers workflow
        ↓
Runner: ubuntu-latest
        ↓
Step 1: actions/checkout@v4 — downloads repo code onto runner
        ↓
Step 2: docker build -t my-docker-app . — builds Docker image
        ↓
Step 3: docker images — lists all images (verify build succeeded)
        ↓
✅ Workflow passes
```

---

> 📅 *Practical #1 — Unit V (GitHub Actions + Docker)*
> 📖 *Reference: Practical_1.docx*
