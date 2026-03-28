# 7. Docker Architecture & Docker Lifecycle

## Docker — Key Components

| Component | What it is |
|-----------|-----------|
| **Docker Client** | Where you type commands (terminal / CLI) |
| **Docker Daemon (dockerd)** | The engine — processes all commands |
| **Docker Images** | Stores all image history/layers |
| **Containers** | Where your project actually runs |
| **Docker Registry (Docker Hub)** | Storage of images |

---

## Docker Architecture Diagram

```
Docker Client (CLI)
        ↕ REST API
Docker Daemon (dockerd)
        ↓
   Docker Host
   ┌──────────────────────────┐
   │  Images   Containers   Networks │
   └──────────────────────────┘
        ↕ Image Pull/Push
   Image Registry
   (Docker Hub / Private)
```

**Flow analogy:**
- Client → **Consumer** (places order)
- Daemon → **Chef** (processes the order)
- Images → **Recipe book** (blueprint)
- Container → **Food** (the actual running app)

---

## Docker Daemon

- The **background service** (engine) that manages everything
- Listens for Docker API requests
- Manages containers, images, networks, volumes
- Called `dockerd`

---

## Docker CLI (Command Line Interface)

- The tool you use to **talk to Docker**
- Sends commands to the Daemon via REST API
- Examples: `docker run`, `docker pull`, `docker ps`

---

## Docker Lifecycle — Build, Ship, Run

```
BUILD → SHIP → RUN
```

| Stage | Action |
|-------|--------|
| **Build** | Create a Docker image (`docker build`) |
| **Ship** | Share image via Docker Hub (`docker push`) |
| **Run** | Start a container from the image (`docker run`) |

### Container States:
```
Created → Running → Paused → Stopped
```

---

## Docker vs Virtual Machine (Summary)

| | Docker Container | Virtual Machine |
|--|----------------|----------------|
| OS | Shares host kernel | Separate Guest OS |
| Boot time | Seconds | Minutes |
| Resource usage | Low | High |
| Isolation | Process-level | Full OS |
| Startup | `docker run` instantly | Boot entire OS |

**Why Docker wins:**
- No Guest OS boot needed
- Containers share the host OS kernel → **lightweight**
- Less resource usage → **faster startup**

---


