# Topic 6: Docker Compose — Building Blocks

**[← Back to Unit 3](README.md)**

## Overview

Learn the YAML structure and components that make up a docker-compose.yml file.

---

## Basic File Structure

```yaml
version: "3.9"

services:
  # Service definitions here
  service_name:
    image: image_name
    ports:
      - "8080:80"

volumes:
  # Named volumes here
  my_volume:

networks:
  # Custom networks here
  my_network:
```

> ⚠️ **YAML Sensitivity:** YAML is **whitespace-sensitive**. Use **2 spaces** for indentation (NOT tabs).

---

## Version

```yaml
version: "3.9"
```

**Compose File Versions:**
| Version | Release | Features |
|---|---|---|
| 3.0 | 2016 | Basic services, volumes, networks |
| 3.5 | 2018 | Better networking, constraints |
| 3.8 | 2020 | Profiles, extended configs |
| 3.9 | 2021 | Latest, recommended |

**Recommendation:** Use `3.8` or `3.9`

---

## Services — Core Concept

**A Service is a BLUEPRINT for containers**, not a container itself.

**Analogy:**
```
Programming:      Class → Instances
Docker Compose:   Service → Containers
```

```yaml
services:
  web:
    image: nginx
# "web" = Service definition (blueprint)
# Docker creates container instances from it
```

### Key Points

- One service = one application component
- Service definition specifies how to build/run containers
- Can scale a service (create multiple instances)
- Services communicate via network
- Each service gets a network alias (service name)

---

## `image` vs `build`

### Using `image` (Pre-built Images)

```yaml
services:
  web:
    image: nginx:latest
    # Pulls from Docker Hub if not locally available
```

**When to Use:**
- Using official/standard images
- Images already built and available
- No custom modifications needed

**Examples:**
```yaml
image: nginx:latest
image: postgres:15
image: redis:7
image: ubuntu:22.04
```

**Format:**
```
image: [REGISTRY/]REPOSITORY[:TAG]

Examples:
nginx                    # Hub, latest
nginx:1.20              # Hub, specific version
myapp:v1.0              # Hub, custom image
docker.io/nginx:latest  # Explicit registry
gcr.io/my-project/app   # Google Container Registry
```

### Using `build` (Build from Dockerfile)

```yaml
services:
  app:
    build: .
    # Builds image from Dockerfile in current directory
```

**When to Use:**
- Using custom application images
- Images need to be built from source
- Different build per environment

**Full Syntax:**
```yaml
services:
  app:
    build:
      context: ./app
      # Where Dockerfile is located
      
      dockerfile: Dockerfile
      # Which Dockerfile to use (default: Dockerfile)
      
      args:
        BUILD_DATE: 2024-01-01
        VERSION: 1.0
      # Build-time arguments
```

**Rebuild When Changes Made:**
```bash
docker-compose up --build
# Forces rebuild of images before starting
```

### Comparison

| Scenario | Use |
|---|---|
| Using PostgreSQL database | `image: postgres:15` |
| Using Redis cache | `image: redis:7` |
| Running your Node.js app | `build: .` |
| Running your Python app | `build: ./backend` |
| Using Nginx reverse proxy | `image: nginx:latest` |

---

## `ports` — Expose Container Ports

**Map container port to host port for external access.**

### Format

```yaml
ports:
  - "HOST_PORT:CONTAINER_PORT"
  - "HOST_PORT:CONTAINER_PORT/PROTOCOL"
```

### Examples

```yaml
ports:
  - "8080:80"         # Host 8080 → Container 80 (HTTP)
  - "8443:443"        # Host 8443 → Container 443 (HTTPS)
  - "3306:3306"       # Host 3306 → Container 3306 (MySQL)
  - "5432:5432"       # Host 5432 → Container 5432 (PostgreSQL)
```

### How It Works

```
User Browser: http://localhost:8080/
         │
         ▼ (Port mapping)
Host Network Stack (port 8080)
         │
         ▼ (Forward to container)
Container Port 80 (Nginx)
         │
         ▼
Response back to browser
```

### Access from Different Locations

```yaml
ports:
  - "8080:80"

# From host machine:
curl http://localhost:8080/

# From another container:
curl http://web:80/        # By service name
# Note: Port 80 is internal, service name works

# From outside (other machine):
curl http://server-ip:8080/
```

### UDP Protocol

```yaml
ports:
  - "8000:8000/udp"    # UDP (not TCP)
  - "8000:8000/tcp"    # Explicit TCP
  - "8000:8000"        # Default TCP
```

---

## `volumes` — Persistent Storage

**Problem:** Containers are **ephemeral** — data lost when container removed.  
**Solution:** Volumes provide persistent storage outside container.

### Named Volumes (Managed by Docker)

```yaml
services:
  db:
    image: mysql:8
    volumes:
      - db-data:/var/lib/mysql
      # Format: volume_name:container_path

volumes:
  # Define volume at root level
  db-data:
```

**Lifecycle:**
```
First run:
  └── Volume created
  └── Data stored in db-data

Container stopped:
  └── Volume persists

Container restarted:
  └── Volume remounted
  └── Data recovered
```

**Benefits:**
- Data persists across container restarts
- Data survives container removal
- Volume managed by Docker
- Easy backup/restore

### Bind Mounts (Mount Host Directory)

```yaml
services:
  app:
    volumes:
      - ./src:/app/src
      # Format: host_path:container_path
```

**Use Cases:**
- Development: Live code sync
- Configuration: Share config files
- Logs: Access container logs from host

**Example (Development):**
```yaml
services:
  node-app:
    image: node:18
    volumes:
      - .:/usr/src/app
    # Changes to host files → reflected in container
    # Useful for development (live reload)
```

### Read-Only Volumes

```yaml
volumes:
  - ./config:/etc/config:ro
  # ":ro" = read-only for container
  # Container can read, but not modify
```

**Use Case:** Configuration files (prevent accidental modification)

### Multiple Volumes

```yaml
services:
  app:
    volumes:
      - app-data:/var/www/app          # Named volume
      - ./config:/etc/app/config       # Bind mount
      - ./logs:/var/log:ro             # Bind mount (read-only)
```

### Volume Types Comparison

| Type | Format | Persistence | Use Case |
|---|---|---|---|
| **Named** | `volume-name:/container/path` | Managed by Docker | Databases, persistent data |
| **Bind** | `/host/path:/container/path` | Host filesystem | Development, config |
| **Read-only** | `...path:ro` | As parent type | Config files, don't modify |
| **Temporary** | `tmpfs:/container/path` | RAM (lost on restart) | Temp cache, performance |

---

## `networks` — Service Communication

**Enable communication between services and external access.**

### Automatic Default Network

```yaml
services:
  web:
    image: nginx
  db:
    image: postgres

# Compose automatically:
# 1. Creates network (usually "app_default")
# 2. Connects both services to it
# 3. Services reach each other by name (DNS)
```

**Service Discovery:**
```
web → needs to connect to db
  └── DNS: "db" resolves to postgres container IP
  └── Connection established
```

### Custom Networks

```yaml
networks:
  frontend:
  backend:

services:
  web:
    networks:
      - frontend      # Only on frontend network
  
  api:
    networks:
      - frontend
      - backend       # On both networks
  
  db:
    networks:
      - backend       # Only on backend network
```

**Communication Matrix:**
```
web ↔ api     ✓ (shared frontend network)
api ↔ db      ✓ (shared backend network)
web ↔ db      ✗ (no shared network)
```

**Benefits:**
- Segmentation (frontend/backend isolation)
- Security (restrict communication paths)
- Microservices patterns (only needed connections)

### Network Drivers

```yaml
networks:
  my_network:
    driver: bridge      # Default, isolated network
    # driver: host      # Use host network
    # driver: overlay   # For swarm mode
```

---

## `depends_on` — Startup Order

**Ensure services start in correct sequence.**

```yaml
services:
  web:
    depends_on:
      - db
  db:
    image: postgres
```

**Startup Order:**
```
1. db service starts
2. web service starts (after db running)
```

### Important Caveat ⚠️

> `depends_on` ensures container **starts**, NOT that service is **ready**.

**Problem Example:**
```yaml
depends_on:
  - db

# db container started but PostgreSQL inside not initialized yet
# web tries to connect → connection refused!
```

### Solution — Health Checks

```yaml
services:
  db:
    image: postgres
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
  
  web:
    depends_on:
      db:
        condition: service_healthy
    # web starts only after db is healthy
```

### Syntax Variations

```yaml
# Simple dependency
depends_on:
  - db
  - cache

# Conditional dependency (v3.0+)
depends_on:
  db:
    condition: service_healthy
  cache:
    condition: service_started
```

---

## `environment` — Environment Variables

**Pass configuration to containers via environment variables.**

### Method 1: Inline in Compose File

```yaml
services:
  app:
    image: my-app
    environment:
      DB_HOST: postgres
      DB_PORT: 5432
      DB_USER: admin
      DB_PASSWORD: secret
      DEBUG: "true"
      LOG_LEVEL: info
```

### Method 2: Using `.env` File

**.env file:**
```
DB_HOST=postgres
DB_PORT=5432
DB_USER=admin
DB_PASSWORD=secret
DEBUG=true
```

**docker-compose.yml:**
```yaml
services:
  app:
    image: my-app
    environment:
      - DB_HOST=${DB_HOST}
      - DB_PORT=${DB_PORT}
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
      - DEBUG=${DEBUG}
```

### Method 3: Using `env_file`

```yaml
services:
  app:
    image: my-app
    env_file:
      - .env
      - .env.production
```

### Default Values

```yaml
environment:
  LOG_LEVEL: ${LOG_LEVEL:-info}
  # If LOG_LEVEL not set, use "info"
```

### Best Practices

```yaml
# ✓ Good - Externalize configuration
environment:
  DATABASE_URL: ${DATABASE_URL}
  API_KEY: ${API_KEY}

# ✗ Bad - Hardcoded secrets
environment:
  DATABASE_URL: postgresql://root:password@localhost/db
  API_KEY: super-secret-key-12345

# ✓ Best - Use .env.example as template
# .env.example (committed to git, no secrets)
# .env (NOT committed, has actual secrets)
```

---

## `restart` — Restart Policy

**Define how container behaves if it exits.**

### Restart Policies

```yaml
restart: no
# Don't restart container
# Best for: Development (want to see errors)

restart: always
# Always restart if exited (even if success)
# Best for: Production services that must always run

restart: on-failure
# Restart only if exited with error code
# Best for: Tasks that should complete successfully

restart: unless-stopped
# Always restart unless explicitly stopped
# Best for: Production (respects manual stop)
```

### On-Failure with Retries

```yaml
restart: on-failure:3
# Maximum 3 restart attempts
# Container stops after 3 failed attempts
```

### Production Example

```yaml
services:
  web:
    image: my-app
    restart: unless-stopped
    # Restarts automatically if crashes
    # But respects docker-compose stop
  
  db:
    image: postgres
    restart: always
    # Always running
    # Automatically recovered if fails
```

---

## Additional Useful Attributes

### Container Name

```yaml
container_name: my-app-container
# Custom container name (default: directory_service_1)
```

### Working Directory

```yaml
working_dir: /usr/src/app
# Working directory inside container
# Commands run from this directory
```

### Command Override

```yaml
command: npm start
# Override default command from image
```

### Resource Limits

```yaml
mem_limit: 512m
# Maximum memory (512MB)

cpus: 1.5
# Maximum CPU (1.5 cores)
```

### User

```yaml
user: www-data
# Run container as specific user
# (instead of default)
```

### Interactive Terminal

```yaml
stdin_open: true
tty: true
# Allocate TTY for interactive use
# Use: docker-compose exec <service> bash
```

---

## Complete Example

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    container_name: web-server
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    depends_on:
      - db
    restart: unless-stopped
    environment:
      - NGINX_HOST=example.com
    networks:
      - app-network

  db:
    image: postgres:15
    container_name: postgres-db
    restart: always
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "admin"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

---

## Next Topic

**[Read about Docker Compose Commands →](07_Docker_Compose_Commands.md)**

Learn essential commands to manage docker-compose applications.
