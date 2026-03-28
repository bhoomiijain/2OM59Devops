# 8. Basic Docker Commands

## Image Commands

```bash
# Check Docker version
docker --version

# Get detailed Docker info
docker info

# Pull an image from Docker Hub
docker pull ubuntu
docker pull nginx
docker pull httpd
docker pull mysql:8

# List all downloaded images
docker images

# View history of an image
docker history httpd

# Delete an image
docker rmi ubuntu

# Force delete (even if used)
docker rmi -f <image_id>

# Delete by image id
docker rmi <image_id>
```

---

## Container Lifecycle Commands

```bash
# Create + Run a container
docker run ubuntu

# Run with interactive terminal
docker run -it ubuntu /bin/bash

# Run in background (detached mode)
docker run -d nginx

# Run with name + port mapping
docker run -d -p 8080:80 --name mynginx nginx

# Run with auto-remove after stop
docker run --rm ubuntu echo "Hello"

# List running containers
docker ps

# List ALL containers (running + stopped)
docker ps -a

# Start a stopped container
docker start <container_id/name>

# Stop a running container
docker stop <container_id/name>

# Restart a container
docker restart <container_id/name>

# Pause a container
docker pause <container_id/name>

# Remove a container
docker rm <container_id>

# Force remove (even if running)
docker rm -f <container_id/name>
```

---

## Container Interaction Commands

```bash
# Open bash terminal inside a running container
docker exec -it <container_id> bash

# Show container output logs
docker logs <container_id>

# Show detailed config in JSON format
docker inspect <container_id>

# Show live CPU + memory usage
docker stats

# Show running processes inside container
docker top <container_id>
```

---

## Port Mapping (`-p` flag)

```
-p <HOST_PORT>:<CONTAINER_PORT>
```

```bash
# Map host port 8080 to container port 80
docker run -d -p 8080:80 --name mynginx nginx

# Access: open browser → localhost:8080
# Flow: Browser → localhost:8080 → Docker Host → Container:80 → Nginx
```

---

## docker run Flags Summary

| Flag | Full Form | Meaning |
|------|-----------|---------|
| `-d` | detached | Run container in background |
| `-it` | interactive + tty | Open interactive terminal |
| `--name` | — | Give container a name |
| `--rm` | — | Auto-remove container when it exits |
| `-p` | publish | Map host port to container port |
| `-e` | env | Set environment variable |
| `-v` | volume | Mount a host directory into container |

---

## Volume Commands

```bash
# Create a persistent storage volume
docker volume create myvolume

# List all volumes
docker volume ls

# Show volume details (& mounting info)
docker volume inspect myvolume

# Delete a volume
docker volume rm myvolume
```

---

## Network Commands

```bash
# List networks (shows ID, name, driver, scope)
docker network ls

# Create a custom network
docker network create mynetwork

# Inspect a network
docker network inspect mynetwork

# Remove a network
docker network rm mynetwork
```

---

## Cleanup Commands

```bash
# Stop ALL running containers
docker stop $(docker ps -aq)

# Remove ALL containers
docker rm $(docker ps -aq)

# Delete ALL images
docker rmi $(docker images -q)

# Clean unused containers, images, networks
docker system prune -a

# Nuclear clean (includes volumes too)
docker system prune -a --volumes
```

---

## Practical Examples from Class

### Example 1 — Nginx with port mapping
```bash
docker pull nginx
docker run -d -p 8080:80 --name mynginx nginx
docker ps
docker stop mynginx
docker start mynginx
docker rm -f mynginx
docker rmi nginx
```

### Example 2 — Ubuntu interactive
```bash
docker run -it ubuntu bash
# Inside container:
echo "Hello from container"
touch /data/hello.txt
exit
```

### Example 3 — Apache (httpd)
```bash
docker pull httpd
docker run -d --name my-apache -p 8080:80 httpd
# Modify the web page inside container:
docker exec -it my-apache bash -c "echo '<h1>Hello from Docker Apache!</h1>' > /usr/local/apache2/htdocs/index.html"
curl http://localhost:8080
docker stop my-apache
docker rm my-apache
```

### Example 4 — MySQL with environment variables
```bash
docker run -d \
  -p 3307:3306 \
  --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=college \
  -e MYSQL_USER=admin \
  -e MYSQL_PASSWORD=admin123 \
  mysql:8

# Connect to MySQL inside container
docker exec -it mysql-test bash
mysql -h 127.0.0.1 -P 3306 -u root -p
```

---


