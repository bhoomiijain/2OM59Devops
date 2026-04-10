# 3. Docker Storage — Volumes, Bind Mounts & CoW

## The Problem

Containers are **ephemeral** — all data inside is lost when a container is removed.

```bash
docker rm mycontainer   # all data inside → gone ❌
```

To persist data across container restarts and deletions, Docker provides:
1. **Volumes** — managed by Docker
2. **Bind Mounts** — managed by you (host path)

---

## 1. Docker Volumes

Volumes are **managed entirely by Docker**, stored outside the container filesystem.

**Default storage location on host:** `/var/lib/docker/volumes/`

### Why use volumes?
- Data persists after container is deleted ✅
- Can be shared between multiple containers ✅
- Easier to back up and migrate ✅
- Best for **production** ✅

### Volume Commands

```bash
# Create a volume
docker volume create myvolume

# List all volumes
docker volume ls

# Inspect (see mount path, driver, creation date)
docker volume inspect myvolume

# Remove a specific volume
docker volume rm myvolume

# Remove ALL unused volumes
docker volume prune
```

### Using a Volume with a Container

```bash
# Syntax
docker run -v <volume_name>:<path_inside_container> image

# Example — attach myvolume to /app/data inside Ubuntu
docker run -d -v myvolume:/app/data --name mycontainer ubuntu

# Write data into it
docker exec -it mycontainer sh -c "echo 'Hello Volume' > /app/data/test.txt"

# Remove container
docker rm -f mycontainer

# Run NEW container with same volume — data is still there!
docker run -it -v myvolume:/app/data ubuntu sh
cat /app/data/test.txt   # → Hello Volume ✅
```

---

## 2. Bind Mounts

A **bind mount** links a specific folder on your **host machine** directly into the container.

**Key difference from volumes:** You specify the exact host path — Docker doesn't manage it.

### Why use bind mounts?
- Great for **development** (live code editing without rebuilding)
- Changes on host are **instantly visible** inside container
- Changes inside container are **instantly visible** on host

```bash
# Syntax
docker run -v <host_path>:<container_path> image

# Linux example
docker run -d \
  -v $(pwd)/html:/usr/share/nginx/html \
  -p 8080:80 \
  nginx

# Windows example
docker run -it \
  --name my_app \
  -e APP_ENV=production \
  -v C:\app\data:/data \
  ubuntu /bin/bash
```

---

## Volumes vs Bind Mounts

| Feature | Volume | Bind Mount |
|---------|--------|------------|
| Managed by Docker | ✅ | ❌ (you manage) |
| Best for production | ✅ | ⚠️ Dev only |
| Host path dependency | ❌ | ✅ (tied to host path) |
| Performance | Good | Good |
| Portability | High | Low |
| Backup/migrate | Easier | Manual |

---

## 3. Copy-on-Write (CoW) Mechanism

Docker images use a **layered filesystem** (OverlayFS).

```
┌────────────────────┐  ← Container writable layer (thin, unique per container)
├────────────────────┤  ← App layer (read-only, shared)
├────────────────────┤  ← Dependencies layer (read-only, shared)
└────────────────────┘  ← Base OS layer (read-only, shared)
```

**How CoW works:**
1. Container **reads** a file → read from the (shared) read-only layer → fast
2. Container **modifies** a file → Docker first **copies it up** to the writable layer → then modifies the copy
3. Original image layers are **never touched**
4. Multiple containers from same image = **one copy of image layers** shared in storage

```bash
# See all layers of an image
docker history nginx

# Check which storage driver Docker is using
docker info | grep "Storage Driver"
# → overlay2  (most common on Linux)
```

---

## 4. Backing Data on Host (MySQL Example)

```bash
# Create a named volume for MySQL data
docker volume create mysql_data

# Run MySQL with volume attached
docker run -d \
  --name mysql_db \
  -e MYSQL_ROOT_PASSWORD=root \
  -v mysql_data:/var/lib/mysql \
  mysql:8

# Even if you delete this container...
docker rm -f mysql_db

# ...data is safe — run a new container with same volume
docker run -d \
  --name mysql_db_new \
  -e MYSQL_ROOT_PASSWORD=root \
  -v mysql_data:/var/lib/mysql \
  mysql:8
```

---

## Hands-on Lab 1 — Volume Persistence (from class)

```bash
# Step 1: Create volume
docker volume create projectdata
docker volume ls
docker volume inspect projectdata

# Step 2: Run container with volume attached
docker run -dit --name project_container -v projectdata:/app/data ubuntu

# Step 3: Write a file inside the container
docker exec -it project_container sh -c "echo 'This is my first volume' > /app/data/report.txt"
docker exec -it project_container cat /app/data/report.txt

# Step 4: Stop and DELETE the container
docker stop project_container
docker rm project_container

# Step 5: Run a NEW container with the SAME volume
docker run -dit --name project_container_new -v projectdata:/app/data ubuntu

# Step 6: Verify data is still there
docker exec -it project_container_new ls /app/data
docker exec -it project_container_new cat /app/data/report.txt
# → This is my first volume  ✅ Data persisted!
```

---

## Hands-on Lab 2 — Full University Portal (from class)

```bash
# Deploy Apache portal with volume + env variable
docker pull httpd
docker volume create portaldata

docker run -dit \
  --name college_portal \
  -p 8080:80 \
  -e ENV=production \
  -v portaldata:/usr/local/apache2/htdocs \
  httpd

# Modify the web page
docker exec -it college_portal bash -c \
  "echo '<h1>Welcome to University Portal - Production Mode</h1>' \
  > /usr/local/apache2/htdocs/index.html"

# Verify
curl http://localhost:8080

# Check logs
docker logs college_portal
docker logs -f college_portal   # follow live

# Cleanup
docker stop college_portal
docker rm college_portal
docker volume rm portaldata
docker rmi httpd
```

---

## Hands-on Lab 3 — Bind Mount with Ubuntu (from class)

```bash
# Create directory on host
mkdir -p ~/app/data       # Linux
# OR: mkdir C:\app\data   # Windows

# Run container with bind mount + env variable
docker run -it \
  --name my_app \
  -e APP_ENV=production \
  -v ~/app/data:/data \
  ubuntu bash

# Inside container:
echo $APP_ENV                          # → production
echo "Hello" > /data/test.txt
ls /data
cat /data/test.txt
exit

# On host — file is visible:
ls ~/app/data
# → test.txt  ✅

# Container deleted — file stays on host
docker rm -f my_app
ls ~/app/data    # → test.txt still there ✅
```

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: New PPT slides 41–53, 63–68 (Basic_Docker_commands + Unit_2)*
