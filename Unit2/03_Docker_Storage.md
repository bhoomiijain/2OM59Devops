# Docker Storage

## Problem

Containers = temporary  
If deleted = all data lost

Need = keep data even after container gone

## Solution 1: Volumes

Docker-managed storage

```
docker volume create mydata
docker run -v mydata:/app/data myimage
```

- Stored in Docker folder
- Data persists
- Can share between containers
- Production use

## Solution 2: Bind Mounts

Link host folder to container

```
docker run -v C:\mydata:/app/data myimage
```

- Changes sync both ways (live)
- Development use
- You control location

## Volumes vs Bind Mounts

| | Volumes | Bind Mounts |
|---|---------|------------|
| Managed by | Docker | You |
| Location | Docker internal | Host path |
| Use | Production | Development |
| Data persists | Yes | Yes |

---
*Need to practice: Create volume and test*

---

## 1. Docker Volumes

A **volume** is a storage location **managed by Docker**, stored outside the container filesystem.

**Key properties:**
- Data persists even after the container is deleted
- Managed by Docker (stored in `/var/lib/docker/volumes/` on host)
- Can be shared between multiple containers
- Best for **production use**

```bash
# Create a volume
docker volume create myvolume

# Use volume when running a container
docker run -d -v myvolume:/data ubuntu

# List all volumes
docker volume ls

# Inspect volume (shows mount path, driver, etc.)
docker volume inspect myvolume

# Remove a volume
docker volume rm myvolume
```

---

## 2. Bind Mounts

A **bind mount** links a **specific folder on your host** directly into the container.

**Key properties:**
- You control the location on the host
- Any changes on host are immediately visible inside the container (and vice versa)
- Great for **development** (live reload without rebuilding image)

```bash
# Syntax
docker run -v <host_path>:<container_path> image

# Linux example
docker run -it -v ~/app/data:/data ubuntu bash

# Windows example
docker run -it -v C:\app\data:/data ubuntu bash
```

---

## Volumes vs Bind Mounts

| Feature | Volumes | Bind Mounts |
|---------|---------|-------------|
| Managed by | Docker | You (host filesystem) |
| Location | Docker internal storage | Any path on host |
| Best for | Production data | Development / live coding |
| Portability | High | Low (tied to host path) |
| Backup | Easier | Manual |
| Shared between containers | Yes | Yes |

---

## Copy-on-Write (CoW) Mechanism

When a container **reads** a file → it reads from the image layer (fast, shared).

When a container **modifies** a file → Docker **copies it up** to the container's writable layer first, then modifies it.

This means:
- Original image layers remain **untouched**
- Multiple containers can share the same image without interfering
- Only changed files take up extra space

```
Image Layer (read-only):   [ base_file.txt ]
                                   ↓ copy-on-write
Container Layer (writable): [ base_file.txt (modified copy) ]
```

---

## Practical Example from Class

```bash
# Create host directory
mkdir -p ~/app/data

# Run container with volume mount + env variable
docker run -it \
  --name my-app \
  -e APP_ENV=production \
  -v ~/app/data:/data \
  ubuntu bash

# Inside container:
echo $APP_ENV                         # → production
touch /data/hello_from_docker.txt     # creates file in mounted volume
echo "Hello" > /data/test.txt
ls /data
cat /data/test.txt
exit

# On host — file is visible:
ls ~/app/data
# → hello_from_docker.txt  test.txt

# Remove container — files on host STILL exist
docker rm -f my-app
ls ~/app/data   # files still there
```

---

## Note on `local` vs `bridge` Driver

```bash
docker volume ls
# DRIVER   VOLUME NAME
# local    myvolume       ← stored on local host

docker network ls
# DRIVER   NETWORK NAME
# bridge   bridge         ← multiple containers share same network
```

- **local** driver → volume stored on the same host
- **bridge** network driver → allows multiple containers to connect using same network

---

 
