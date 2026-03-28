# Docker Networking

## Why needed?

Containers isolated by default = can't talk to each other

Need networking to:
- Connect containers together
- Access from browser
- Talk to database
- etc

## Network Types

**Bridge** (default)
- Containers on same bridge can talk
- When you do `docker run` = bridge network
- List: `docker network ls`

**Host**
- Container uses host's network directly
- No port mapping
- Less isolation = faster

**Overlay**
- Multi-host (Swarm/Kubernetes)
- For distributed stuff

**None**
- No network access at all
- Maximum isolation

## Port Mapping

```
-p HOST_PORT:CONTAINER_PORT
```

Example:
```
docker run -p 8080:80 nginx
```

Means: Host 8080 → Container 80

Can access from browser: `localhost:8080`

## Container Communication

Two containers on same network = can talk using container names

```
docker network create mynet

docker run -d --name app1 --network mynet nginx
docker run -d --name app2 --network mynet python-app

# app2 can reach app1 using "app1" as hostname
```

---
*Practice: Create custom network and connect containers*

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

