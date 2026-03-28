# Container Runtime & Namespaces

## Container Runtime?

Software that creates/starts/stops containers. Takes image → makes it run

**Basically:** Docker, containerd, CRI-O, Podman

**Docker** = most used one

**What it does:**
- Runs container processes
- Isolates stuff
- Sets resource limits
- Manages lifecycle

---

## Namespaces (Process Isolation)

Container linux feature = isolate processes

**Why?** So containers think they're separate from each other and host

**Types of namespaces:**
- PID - process IDs (each container sees different PID 1)
- Network - separate network (own IP, ports)
- Mount (mnt) - separate file systems
- IPC - inter-process communication
- UTS - hostname stuff
- User - different users in containers

**Example:** Container A thinks PID 1 is its app, but host sees it differently

Each container = isolated world

*Note: Didn't fully understand UTS namespace - ask in next class*

**Namespaces** are Linux kernel features that **partition system resources** so each container sees only its own environment.

> "Namespaces create the illusion of a separate system for each container."

**Isolation is achieved using Linux namespaces — each container runs as if it is alone on the system**, even though multiple containers share the same OS kernel.

---

## Types of Namespaces

### 1. PID Namespace (Process Isolation)
- Each container has its **own process tree**
- PIDs inside container **start from 1**
- A container **cannot see** host or other container processes

```
Container view:        Host view:
PID 1 → app           PID 1547 → app
PID 2 → worker        PID 1548 → worker
```

### 2. Network Namespace
- Each container gets its **own IP address and ports**
- Containers can run on the **same port without conflict**
  - e.g., two containers both using port 8080 — no conflict!

### 3. Mount Namespace (Volume/Filesystem)
- Each container has its **own filesystem view**
- File changes inside a container **do not affect the host**

### 4. UTS Namespace
- Allows each container to have its **own hostname**

### 5. User Namespace (Advanced)
- Maps container user to a **non-root user** on host
- **Improves security** significantly

---

## Summary Table

| Namespace | What it isolates |
|-----------|-----------------|
| PID | Process IDs |
| Network | IP address, ports |
| Mount | Filesystem |
| UTS | Hostname |
| User | User IDs / permissions |

> **Remember:** Namespaces = **Isolation**. cgroups = **Resource control**

---

