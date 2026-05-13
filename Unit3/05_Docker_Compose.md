# Topic 5: Docker Compose

**[← Back to Unit 3](README.md)**

## Overview

Docker Compose is a tool for defining and managing multi-container Docker applications using a simple YAML configuration file.

---

## Definition

**Docker Compose:**
> Docker Compose is a tool to define and manage multi-container Docker applications using a simple YAML configuration file. Instead of running containers individually with separate commands, you define the entire application stack declaratively in one file.

**Key Features:**
- Define services in YAML
- Manage multiple containers together
- Network services automatically
- Persistent data with volumes
- Environment configuration
- Single command to start/stop everything

---

## What Problem Does Docker Compose Solve?

### Without Docker Compose (Manual Approach)

**Modern applications need multiple containers:**

```
Web Application Stack:
├── Frontend (React)
├── Backend API (Node.js)
├── Database (PostgreSQL)
├── Cache (Redis)
└── Message Queue (RabbitMQ)
```

**Manual Commands Required:**

```bash
# Start database
docker run -d -p 5432:5432 \
  -e POSTGRES_DB=mydb \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret \
  -v db-data:/var/lib/postgresql/data \
  --name postgres \
  postgres:13

# Start Redis cache
docker run -d -p 6379:6379 \
  --name redis \
  redis:7

# Start backend API
docker run -d -p 8000:8000 \
  -e DB_HOST=postgres \
  -e DB_USER=admin \
  -e DB_PASSWORD=secret \
  -e REDIS_HOST=redis \
  --link postgres \
  --link redis \
  --name backend \
  my-backend:latest

# Start frontend
docker run -d -p 3000:3000 \
  -e API_URL=http://backend:8000 \
  --link backend \
  --name frontend \
  my-frontend:latest

# ... and many more commands
```

### Problems with Manual Approach

| Problem | Impact |
|---|---|
| **Too many commands** | Hard to remember all flags and parameters |
| **Hard to reproduce** | Different developers make different choices |
| **Networking complexity** | Manual IP/hostname management |
| **Dependency management** | Hard to ensure startup order |
| **Error-prone** | One typo breaks everything |
| **Difficult to shutdown** | Need to manually stop each container |
| **Not version-controlled** | Commands live in notes, not repo |
| **Team inconsistency** | Everyone sets up differently |

### With Docker Compose (Solution)

```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

  redis:
    image: redis:7

  backend:
    image: my-backend:latest
    ports:
      - "8000:8000"
    environment:
      DB_HOST: postgres
      REDIS_HOST: redis
    depends_on:
      - postgres
      - redis

  frontend:
    image: my-frontend:latest
    ports:
      - "3000:3000"
    environment:
      API_URL: http://backend:8000
    depends_on:
      - backend

volumes:
  db-data:
```

**Single command to start everything:**

```bash
docker-compose up -d
# All services start automatically
# Networking configured
# Dependencies respected
# Volumes created
# Environment variables set
```

---

## How Docker Compose Solves the Problems

### (A) Multi-Container Management

```
Before: Run dozens of commands
After:  Single docker-compose up command

Composefile as source of truth:
├── What services run?
├── What images they use?
├── How they communicate?
├── What resources they need?
└── Everything in one file
```

### (B) Reproducibility

**"Works on my machine" problem solved:**

```
Developer 1's Laptop
├── docker-compose.yml
├── Pull code
├── docker-compose up
└── All containers start identically

Developer 2's Laptop
├── docker-compose.yml
├── Pull code
├── docker-compose up
└── All containers start identically

Production Server
├── docker-compose.yml
├── Pull code
├── docker-compose up
└── All containers start identically
```

**Result:** Identical environment everywhere

### (C) Simplified Networking

**Without Compose:**
```bash
# Get database container IP
docker inspect postgres | grep IPAddress
# Output: "IPAddress": "172.17.0.2"

# Connect backend to it
docker run -e DB_HOST=172.17.0.2 backend
```

**Problems:**
- Manual IP management
- IPs change when containers restart
- Error-prone

**With Compose:**
```yaml
services:
  postgres:
    image: postgres
  backend:
    image: my-backend
    # Automatically connects to "postgres" by hostname!
```

**Benefits:**
- Services reach each other by name (DNS discovery)
- Automatic network creation
- No manual IP management
- Survives container restarts

### (D) Dependency Management

```yaml
services:
  backend:
    depends_on:
      - postgres
```

**Ensures:**
- `postgres` starts before `backend`
- Services start in correct order
- Reduces race conditions

> ⚠️ **Note:** `depends_on` ensures startup **order**, not service **readiness**

### (E) Scalability

```bash
# Run 3 instances of web service
docker-compose up --scale web=3
# Creates: web_1, web_2, web_3
```

### (F) Cleaner Configuration

**Before (Dockerfile approach):**
```bash
docker run -d -p 8080:80 \
  -e WORDPRESS_DB_HOST=db:3306 \
  -e WORDPRESS_DB_USER=admin \
  -e WORDPRESS_DB_PASSWORD=secret \
  -v wordpress-data:/var/www/html \
  --depends-on mysql \
  --restart always \
  wordpress:latest
```

**After (Docker Compose):**
```yaml
services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: admin
      WORDPRESS_DB_PASSWORD: secret
    volumes:
      - wordpress-data:/var/www/html
    depends_on:
      - db
    restart: always
```

**Benefits:**
- Readable and maintainable
- Version-controlled
- Self-documenting
- Shareable across team

---

## Docker vs Docker Compose

| Aspect | Docker | Docker Compose |
|---|---|---|
| **Scope** | Manages individual containers | Manages groups of containers |
| **Configuration** | Command-line arguments | YAML file |
| **Commands** | `docker run`, `docker stop` | `docker-compose up`, `docker-compose down` |
| **Networking** | Manual with `--link` flag | Automatic service discovery |
| **Use Case** | Single container apps | Multi-container applications |
| **Complexity** | One container at a time | Multiple containers as system |
| **Best For** | Simple deployments | Development, testing, staging |

---

## Docker Compose Design Principles

### 1. Declarative Configuration

**Specify WHAT, not HOW:**

```yaml
# Declarative: WHAT we want
services:
  web:
    image: nginx
    ports:
      - "8080:80"

# Imperative approach (less good)
# docker run -d -p 8080:80 nginx
# docker exec web ...
# docker inspect web...
# etc.
```

### 2. Infrastructure as Code (IaC)

**Infrastructure defined in code:**
- Version-controlled
- Reviewable (pull requests)
- Trackable (git history)
- Reproducible

### 3. Reproducibility

**Same config → Same environment everywhere**

```
laptop → staging → production
All use same docker-compose.yml
All environments identical
```

### 4. Automation

**Eliminate manual steps:**
- No manual container starting
- No manual networking setup
- No manual volume creation
- Single command: `docker-compose up`

---

## File Organization

### Single File (Simple Projects)

```
project/
├── docker-compose.yml
├── Dockerfile
├── app.py
└── requirements.txt
```

### Multiple Files (Complex Projects)

```
project/
├── docker-compose.yml          (main file)
├── docker-compose.override.yml (development overrides)
├── docker-compose.prod.yml     (production config)
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py
└── database/
    └── init.sql
```

---

## Use Cases for Docker Compose

### ✅ Perfect For:
- Local development
- Testing multi-container apps
- Staging environments
- CI/CD pipelines
- Demo/prototype systems
- Small production deployments

### ❌ Not Ideal For:
- Large-scale production (use Kubernetes)
- High availability needs (use Kubernetes)
- Auto-scaling requirements (use Kubernetes)
- Multiple hosts (use Kubernetes)

---

## Comparison: Docker Compose vs Kubernetes

| Aspect | Docker Compose | Kubernetes |
|---|---|---|
| **Complexity** | Simple | Complex |
| **Hosts** | Single machine | Multiple machines |
| **Scale** | Dozens of containers | Thousands of containers |
| **Auto-scaling** | Manual | Automatic |
| **Self-healing** | No | Yes |
| **Rolling updates** | Manual | Automatic |
| **Learning curve** | Easy (days) | Hard (weeks) |
| **When to use** | Development, small projects | Production, enterprise |

---

## Next Topic

**[Read about Docker Compose Building Blocks →](06_Docker_Compose_Building_Blocks.md)**

Learn the YAML structure and key components of docker-compose.yml files.
