# 2. Docker Networking

## Why Docker Networking?

By default, containers are **isolated** — they can't talk to each other or the outside world unless configured.

Docker networking allows:
- Containers to communicate with each other
- Containers to be accessed from host/browser
- Controlled isolation between services

---

## Types of Docker Networks

### 1. Bridge Network (Default)
- Created automatically when Docker is installed
- Containers on the same bridge network **can talk to each other**
- Used when running multiple containers on a **single host**
- Default network when you run `docker run` without specifying a network

```bash
docker network ls
# → bridge, host, none (default networks)
```

### 2. Host Network
- Container **shares the host's network stack** directly
- No port mapping needed — container uses host ports directly
- **Less isolation** but better performance

```bash
docker run --network host nginx
```

### 3. Overlay Network
- Used for **multi-host** networking (Docker Swarm / Kubernetes)
- Containers on different hosts can communicate
- Encrypted by default

### 4. None Network
- Container has **no network access** at all
- Maximum isolation

---

## DNS Inside Docker

When containers are on the **same custom network**, Docker provides:
- **Automatic DNS resolution** by container name
- You can use container names as hostnames

```bash
# Create a custom network
docker network create mynetwork

# Run two containers on same network
docker run -d --name webapp --network mynetwork nginx
docker run -d --name db --network mynetwork mysql:8

# webapp can reach db using hostname "db"
# e.g., db connection string: mysql://db:3306/mydb
```

---

## Port Mapping (Host ↔ Container)

```
-p <HOST_PORT>:<CONTAINER_PORT>
```

| Part | Description |
|------|-------------|
| HOST_PORT | Port on your laptop/server |
| CONTAINER_PORT | Port where the app runs inside container |

```bash
# Map host 8080 → container 80
docker run -d -p 8080:80 nginx

# Multiple port mappings
docker run -d -p 8080:80 -p 443:443 nginx

# Map host 3307 → container 3306 (MySQL)
docker run -d -p 3307:3306 --name mysql-test -e MYSQL_ROOT_PASSWORD=root123 mysql:8
```

**Access flow:**
```
Browser → localhost:8080 → Docker Host → Container:80 → Nginx
```

---

## Linking Containers

Older way (use custom networks instead):
```bash
docker run --link db:database webapp
```

**Modern approach — use custom bridge networks:**
```bash
docker network create appnet
docker run -d --name db --network appnet mysql:8
docker run -d --name web --network appnet -p 8080:80 nginx
```

---

## Network Commands

```bash
# List all networks (ID, name, driver, scope)
docker network ls

# Create a custom network
docker network create mynetwork

# Run a container on a specific network
docker run -d --network mynetwork --name myapp nginx

# Inspect network details
docker network inspect mynetwork

# Remove a network
docker network rm mynetwork

# Connect a running container to a network
docker network connect mynetwork <container_id>

# Disconnect container from network
docker network disconnect mynetwork <container_id>
```

---

## Practice Example from Class

```bash
# Create apache container accessible on host
docker pull httpd
docker run -d --name apache-web -p 8081:80 httpd

# Verify
docker ps
# Open browser: localhost:8081
```

---

