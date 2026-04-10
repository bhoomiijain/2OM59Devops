# 9. Docker Object Types — Container, Image, Network, Volume

> This topic covers the **four core object types** Docker manages. Understanding these is essential before working with Dockerfiles or advanced networking.

---

## The Four Docker Object Types

| Object | What it is | Created by |
|--------|-----------|------------|
| **Image** | Read-only blueprint/template | `docker build` or `docker pull` |
| **Container** | Running instance of an image | `docker run` |
| **Network** | Communication channel between containers | `docker network create` |
| **Volume** | Persistent storage attached to containers | `docker volume create` |

---

## 1. Image

- A **read-only, layered package** containing app + OS + dependencies
- Built from a Dockerfile or pulled from a registry
- **Immutable** — never changes after creation
- Multiple containers can be created from one image

```bash
docker images                    # list all images
docker pull nginx                # pull from Docker Hub
docker rmi nginx                 # remove image
docker inspect nginx             # detailed metadata
docker history nginx             # view all layers
```

---

## 2. Container

- A **running (or stopped) instance** of an image
- Has its own writable layer on top of the image
- Isolated via **namespaces** (PID, Network, Mount, etc.)
- Resource-limited via **cgroups**
- Ephemeral by default — data is lost when removed

```bash
docker run -d --name myapp nginx          # create + start
docker ps                                  # list running
docker ps -a                               # list all (including stopped)
docker stop myapp                          # stop
docker start myapp                         # restart
docker rm myapp                            # delete container
docker inspect myapp                       # detailed JSON info
docker logs myapp                          # view output logs
docker logs -f myapp                       # follow logs live
docker stats myapp                         # live CPU/memory usage
docker top myapp                           # running processes inside
```

### Container Lifecycle States
```
Created → Running → Paused → Stopped → Removed
```

### Container Interaction Commands
```bash
# Open terminal inside container
docker exec -it <container_id> bash

# Attach to container's main process
docker attach <container_id>

# Copy file FROM container TO host
docker cp <container_id>:/path/file.txt ~/Desktop/

# Copy file FROM host TO container
docker cp ~/Desktop/file.txt <container_id>:/path/

# Kill container (force stop)
docker kill <container_id>

# Remove all stopped containers
docker container prune
```

---

## 3. Network

- Controls **how containers communicate** with each other and the outside world
- Three default networks are created automatically: `bridge`, `host`, `none`

```bash
docker network ls                          # list networks
docker network create mynetwork           # create custom network
docker network inspect mynetwork          # show details
docker network rm mynetwork               # delete
docker network connect mynetwork myapp    # add container to network
docker network disconnect mynetwork myapp # remove container from network
```

**Connecting container to a network at run time:**
```bash
docker run -d --network mynetwork --name myapp nginx
```

---

## 4. Volume

- **Persistent storage** that survives container deletion
- Managed by Docker, stored at `/var/lib/docker/volumes/` on host
- Can be shared between multiple containers

```bash
docker volume create myvolume             # create
docker volume ls                          # list
docker volume inspect myvolume           # show details (mount path etc.)
docker volume rm myvolume                # delete
docker volume prune                       # delete all unused volumes
```

**Using a volume with a container:**
```bash
docker run -d -v myvolume:/app/data --name myapp ubuntu
```

---

## Docker Layering & Filesystem (Union FS)

Docker uses **OverlayFS (Union Filesystem)** to stack image layers:

```
┌─────────────────────────────┐  ← Container writable layer (thin)
├─────────────────────────────┤  ← App layer (read-only)
├─────────────────────────────┤  ← Dependencies layer (read-only)
├─────────────────────────────┤  ← Runtime layer (read-only)
└─────────────────────────────┘  ← Base OS layer (read-only)
```

**Copy-on-Write (CoW):**
- Containers share image layers (read-only)
- When a file is modified → Docker **copies it up** to the writable layer
- Original image layers are never touched
- Multiple containers from same image = only one copy of image layers in storage

```bash
# See layer info
docker history nginx
docker info | grep "Storage Driver"
```

---

> 📅 *Notes from Class — Unit I*  
> 📖 *Reference: Slides + New PPT (Unit_2 slides 36–44)*
