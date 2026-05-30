# Practical 5 — CI Pipeline: Build & Push Docker Image to Docker Hub

## Problem Statement
Set up a CI pipeline using GitHub Actions that:
- Triggers on every push to `main`
- Checks out the repository
- Logs into Docker Hub using **GitHub Secrets**
- Builds the Docker image from the `Dockerfile`
- Tags it as `username/app-ci:latest`
- Pushes it to Docker Hub

Also explain the role of **environment variables** and **secrets** in this workflow.

---

## Pre-requisites

### 1. Create Docker Hub Access Token
1. Log in to [hub.docker.com](https://hub.docker.com)
2. Go to: **Account Settings → Security → New Access Token**
3. Copy the token — you won't see it again

### 2. Add GitHub Secrets
In your GitHub repo:
`Settings → Secrets and variables → Actions → New repository secret`

| Secret Name | Value |
|-------------|-------|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | The access token you just created |

> **Why secrets?** Secrets are encrypted — they never appear in logs. Anyone who could read your YAML would never see the actual values.

---

## Project Files

Reuse the same Flask app from Practical 1:

**`app.py`**
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from Docker CI/CD!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

**`requirements.txt`**
```
Flask
```

**`Dockerfile`**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

---

## Workflow File — `.github/workflows/docker-ci.yml`

```yaml
name: Docker CI — Build and Push to Docker Hub

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker Image
        run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/app-ci:latest .

      - name: Push Docker Image to Docker Hub
        run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/app-ci:latest
```

---

## Workflow Explained — Step by Step

```
Push code to main branch
        ↓
GitHub Actions triggers workflow
        ↓
Runner: ubuntu-latest (fresh Linux VM)
        ↓
Step 1: Checkout — clone your repo onto the runner
        ↓
Step 2: Log in to Docker Hub using secrets (no password in YAML)
        ↓
Step 3: Build image — tagged as username/app-ci:latest
        ↓
Step 4: Push image → Docker Hub registry
        ↓
✅ Image visible at hub.docker.com/r/username/app-ci
```

---

## Role of Environment Variables & Secrets

### Environment Variables
- Key-value pairs available to all steps in a workflow
- Defined at workflow, job, or step level
- Used for non-sensitive config (e.g., app name, port, environment)

```yaml
env:
  APP_NAME: my-flask-app
  APP_PORT: 5000

jobs:
  build:
    steps:
      - name: Print env
        run: echo "App is $APP_NAME on port $APP_PORT"
```

### Secrets
- **Encrypted** key-value pairs stored in GitHub
- Never visible in logs — shown as `***`
- Used for **sensitive data**: passwords, tokens, API keys
- Accessed via `${{ secrets.SECRET_NAME }}`

```yaml
# CORRECT — use secrets for credentials
username: ${{ secrets.DOCKERHUB_USERNAME }}
password: ${{ secrets.DOCKERHUB_TOKEN }}

# WRONG — never hardcode credentials
username: bhoomiijain
password: mypassword123
```

### Why Secrets Over Plain Env Variables?

| | Environment Variable | Secret |
|--|---------------------|--------|
| Visible in logs | ✅ Yes | ❌ Masked/hidden |
| Encrypted storage | ❌ No | ✅ Yes |
| Use for | Config, non-sensitive | Passwords, tokens, keys |
| Accessible from forks | ✅ Yes | ❌ No (protected) |

---

## Verify on Docker Hub

After workflow runs successfully:
1. Go to [hub.docker.com](https://hub.docker.com)
2. Navigate to your repository: `username/app-ci`
3. You should see the `latest` tag with a recent push timestamp ✅

---

## Pull & Test the Published Image

```bash
# Pull from Docker Hub
docker pull yourusername/app-ci:latest

# Run it
docker run -p 5000:5000 yourusername/app-ci:latest

# Open browser → http://localhost:5000
# Output: Hello from Docker CI/CD!
```

---

> 📅 *Practical #5 — Unit V (CI Pipeline → Docker Hub Push)*
> 📖 *Reference: Practical_no_5.docx*
