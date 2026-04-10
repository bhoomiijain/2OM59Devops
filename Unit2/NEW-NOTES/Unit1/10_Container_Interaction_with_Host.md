# 10. Container Interaction with Host — exec, cp, attach

> This section covers how to **work inside a running container** and **transfer files** between container and host.

---

## docker exec — Run Commands Inside Container

`docker exec` lets you run any command inside an **already running** container.

```bash
# Open an interactive bash terminal
docker exec -it <container_id> bash

# Open sh (for Alpine-based images that don't have bash)
docker exec -it <container_id> sh

# Run a single command without entering terminal
docker exec -it <container_id> ls /
docker exec -it <container_id> cat /etc/os-release

# Run multiple commands
docker exec -it <container_id> sh -c "echo 'Hello Docker' > /data/test.txt"
```

**Flags:**
- `-i` = interactive (keep stdin open)
- `-t` = allocate a terminal (TTY)
- `-d` = detached (run in background)

---

## docker attach — Attach to Main Process

`docker attach` connects your terminal directly to the **main process (PID 1)** of the container.

```bash
docker attach <container_id>
```

> ⚠️ If you press `Ctrl+C` here, it **stops the container** (kills PID 1). Use `Ctrl+P Ctrl+Q` to detach safely.

**Difference:**

| | `docker exec` | `docker attach` |
|--|--------------|----------------|
| Creates new process? | ✅ Yes | ❌ No (joins main process) |
| Safe to exit? | ✅ `exit` or `Ctrl+D` | ⚠️ `Ctrl+P Ctrl+Q` |
| Use for | Running commands, debugging | Monitoring main process output |

---

## docker cp — Copy Files Between Container and Host

Copy files **both ways** — no need to enter the container.

```bash
# FROM container → TO host
docker cp <container_id>:/path/inside/container /path/on/host

# FROM host → TO container
docker cp /path/on/host <container_id>:/path/inside/container
```

**Examples:**
```bash
# Copy a log file from container to Desktop
docker cp mycontainer:/app/app.log ~/Desktop/app.log

# Copy a config file from host into container
docker cp ~/Desktop/config.json mycontainer:/app/config.json

# Windows examples
docker cp mycontainer:/data/test.txt C:\Users\HP\Desktop\
docker cp C:\Users\HP\Desktop\test.txt mycontainer:/data/sample.txt
```

---

## Hands-on Lab — Container Interaction (from class)

```bash
# Step 1: Run a container
docker run -d --name myubuntu ubuntu sleep 300

# Step 2: Enter it
docker exec -it myubuntu /bin/bash

# Step 3: Create files inside
mkdir /data
echo "Hello Docker" > /data/test.txt
exit

# Step 4: Attach (then detach safely)
docker attach myubuntu
# Press: Ctrl+P then Ctrl+Q   (detach without stopping)

# Step 5: Copy file from container to host
docker cp myubuntu:/data/test.txt ~/Desktop/

# Step 6: Copy file from host to container
docker cp ~/Desktop/test.txt myubuntu:/data/sample.txt

# Step 7: Verify both files exist inside container
docker exec -it myubuntu ls /data
docker exec -it myubuntu cat /data/sample.txt
```

---

## Class Task — Container Interaction Assignment

```bash
# Task: Create /project directory, add files, copy to Desktop

# 1. Open terminal inside running container
docker exec -it <container_id> bash

# 2. Create directory and file
mkdir /project
echo "This is our container interaction with our host machine." > /project/report.txt
cat /project/report.txt
exit

# 3. Copy report.txt to Desktop
docker cp <container_id>:/project/report.txt C:\Users\HP\Desktop\

# 4. Create another file inside container
docker exec -it <container_id> sh -c \
  "echo 'This is an additional file inside the same container.' > /project/notes.txt"

# 5. Verify both files exist
docker exec -it <container_id> ls /project
docker exec -it <container_id> cat /project/notes.txt
```

---

## Quick Reference — All Interaction Commands

```bash
# Enter container
docker exec -it <id> bash

# Run command directly
docker exec -it <id> sh -c "command here"

# Follow logs live
docker logs -f <id>

# Copy container → host
docker cp <id>:/container/path /host/path

# Copy host → container
docker cp /host/path <id>:/container/path

# Live resource usage
docker stats <id>

# Running processes inside
docker top <id>

# Full config in JSON
docker inspect <id>
```

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: New PPT slides 39–44 (Unit_2__1_ + Basic_Docker_commands)*
