# 2. Docker Networking — Complete Guide

## Why Docker Networking?

By default, containers are **isolated** — they can't talk to each other or the outside world without configuration.

Docker networking allows:
- Containers to communicate with each other
- Containers to be reached from browser/host
- Controlled isolation between services
- DNS-based service discovery (by container name)

---

## Default Networks (Auto-created by Docker)

```bash
docker network ls
# NETWORK ID   NAME      DRIVER    SCOPE
# abc123       bridge    bridge    local
# def456       host      host      local
# ghi789       none      null      local
```

---

## A. Bridge Network (Default)

- **Default** network when you run any container
- Containers on the same bridge can communicate **via IP**
- Best for **standalone containers on a single host**
- Custom bridge networks also support **DNS by name**

```bash
# List networks
docker network ls

# Inspect the default bridge
docker network inspect bridge

# Create a CUSTOM bridge network
docker network create my_bridge_net

# Run containers on custom bridge
docker run -d --name container1 --network my_bridge_net nginx
docker run -d --name container2 --network my_bridge_net alpine sleep 300

# Test: container2 can ping container1 by NAME
docker exec -it container2 ping container1
```

> **Important:** DNS by container name **only works on custom (user-defined) bridges**, not on the default `bridge` network.

---

## B. Host Network

- Container **shares the host machine's network stack directly**
- No separate IP address for container
- No need for `-p` port mapping — app listens on host ports directly
- **Linux only** (not supported on Docker Desktop for Windows/Mac)
- Faster performance, but **less isolation**

```bash
docker run -d --network host nginx
curl http://localhost      # works directly — no port mapping needed
```

---

## C. Overlay Network

- Used in **Docker Swarm** (multi-host setup)
- Connects containers running on **different physical/virtual hosts**
- Encrypted by default

```bash
# Initialize Docker Swarm first
docker swarm init

# Create overlay network
docker network create -d overlay my_overlay

# Deploy a service using overlay
docker service create --name web --network my_overlay -p 8080:80 nginx
```

---

## D. None Network

- Container has **zero network access**
- Maximum isolation, no connectivity at all

```bash
docker run --network none ubuntu
```

---

## DNS Inside Docker

Docker has a **built-in DNS server** for user-defined networks.

- Containers on the same **custom network** can reach each other by **container name**
- Works **only** on user-defined bridge networks (not the default bridge)

```bash
# Create a custom network
docker network create app_net

# Run two containers on it
docker run -d --name backend --network app_net nginx
docker run -it --name client --network app_net alpine sh

# Inside client container — reach backend by NAME:
ping backend
# → This works because Docker DNS resolves "backend" to its IP
```

---

## Port Mapping (Host ↔ Container)

Without `-p`, containers are **not accessible from browser or host**.

```
Syntax: -p <HOST_PORT>:<CONTAINER_PORT>
```

```bash
# Map host 8080 → container 80
docker run -d -p 8080:80 --name mynginx nginx

# Multiple ports
docker run -d -p 8080:80 -p 443:443 nginx

# MySQL: host 3307 → container 3306
docker run -d -p 3307:3306 \
  --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=root123 \
  mysql:8
```

**Internal flow:**
```
Browser → localhost:8080 → Docker Host → Container:80 → Nginx
```

---

## Linking Containers (Legacy — Deprecated)

The old `--link` way. **Don't use this** — use custom networks instead.

```bash
# OLD WAY (deprecated)
docker run -d --name db nginx
docker run -it --name app --link db:database alpine sh

# MODERN WAY (use custom network)
docker network create appnet
docker run -d --name db --network appnet nginx
docker run -it --name app --network appnet alpine sh
# Inside app: ping db    ← works by DNS
```

---

## Network Commands — Full Reference

```bash
# List all networks
docker network ls

# Create a network
docker network create mynetwork

# Inspect network
docker network inspect mynetwork

# Delete network
docker network rm mynetwork

# Connect a running container to a network
docker network connect mynetwork mycontainer

# Disconnect
docker network disconnect mynetwork mycontainer
```

---

## Complete Multi-Container Example (from class)

```bash
# 1. Create shared network
docker network create app_bridge

# 2. Create volume for MySQL data
docker volume create mysql_data

# 3. Run MySQL with network + volume
docker run -d \
  --name mysql_db \
  --network app_bridge \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=myapp \
  -v mysql_data:/var/lib/mysql \
  mysql:8

# 4. Run backend connected to same network
docker run -d \
  --name backend_app \
  --network app_bridge \
  -e DB_HOST=mysql_db \
  node:18-alpine sh -c "apk add curl && sleep 300"

# 5. Test connectivity (backend can reach mysql_db by name)
docker exec -it backend_app ping mysql_db

# 6. Run frontend on same network + expose to host
docker run -d \
  --name frontend \
  --network app_bridge \
  -p 8080:80 \
  nginx
```

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: New PPT slides 57–63, 66–72 (Basic_Docker_commands + Unit_2)*
