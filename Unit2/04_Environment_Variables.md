# 4. Environment Variables in Docker

## What are Environment Variables?

**Environment variables** are **key-value pairs** used by applications to:
- Read configuration
- Store secrets (usernames, passwords)
- Control application behavior **without changing the image**

> They allow us to **configure containers dynamically** — same image, different behavior.

---

## Why Use Them?

| Without `-e` | With `-e` |
|-------------|----------|
| Configuration must be hard-coded in image | Same image → different behavior in dev/prod |
| Need to rebuild image for every change | Secure & flexible configuration |
| Less flexible | Widely used in microservices & cloud deployments |

---

## Setting Environment Variables

### Method 1 — Using `-e` flag with `docker run`

```bash
# Single variable
docker run -e MY_VAR=value httpd env

# Named container with env variable
docker run -it -e MY_NAME=Bhoomi ubuntu bash

# Inside container:
echo $MY_NAME   # → Bhoomi
```

> **Note:** `MY_NAME` exists **only inside the container** — host system is NOT affected.

---

### Method 2 — Multiple Environment Variables

```bash
docker run \
  -e APP_ENV=production \
  -e APP_VERSION=1.0 \
  nginx
```

---

### Method 3 — MySQL Example (required vars)

```bash
docker run -d \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=college \
  -e MYSQL_USER=admin \
  -e MYSQL_PASSWORD=admin123 \
  mysql:8
```

> ⚠️ Without these variables, the MySQL container **will fail to start**.

---

### Method 4 — Passing from Host System

```bash
# Step 1: Set variable on host
export APP_PORT=8080
export API_KEY=12345

# Step 2: Pass to container (Docker reads from host env)
docker run -e APP_PORT nginx env
```

---

### Method 5 — Using `--env-file` (Clean & Professional)

Create a `.env` file:
```
DB_HOST=localhost
DB_USER=root
DB_PASS=secret
APP_ENV=production
```

Run container with the file:
```bash
docker run --env-file .env myapp
```

> Best practice for **production** — keeps secrets out of command history.

---

## Checking Environment Variables Inside a Container

```bash
# Open container terminal
docker exec -it <container_id> bash

# View all env variables
env

# Check specific variable
echo $MY_NAME
echo $APP_ENV
```

---

## Practical Example from Class

```bash
# Run Ubuntu with env variable + volume + name
docker run -it \
  --name my-app \
  -e APP_ENV=production \
  -v ~/app/data:/data \
  ubuntu bash

# Inside container:
echo $APP_ENV          # → production
echo "Hello" > /data/test.txt
cat /data/test.txt
exit

# On host:
echo $APP_ENV          # → empty (host not affected) ✅
```

---

## Important Notes

- Variables are scoped **only to the container**
- Host system variables are NOT changed
- Use **UPPER_CASE** naming convention for env variables (e.g., `MY_NAME`, `APP_ENV`)
- For secrets, prefer `--env-file` or Docker Secrets (advanced)

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: Slides (Slides 11–17) + Handwritten notes (Pages 15–18)*
