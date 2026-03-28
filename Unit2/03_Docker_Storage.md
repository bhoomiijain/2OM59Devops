# 3. Docker Storage — Volumes & Bind Mounts

## The Problem with Container Storage

By default, when a container is **deleted**, all data inside it is **lost**.

> Containers are ephemeral (temporary) by nature.

To **persist data**, Docker provides two main storage mechanisms:
1. **Volumes** (managed by Docker)
2. **Bind Mounts** (managed by you)

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
ls ~/app/data   # files still there ✅
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

 
