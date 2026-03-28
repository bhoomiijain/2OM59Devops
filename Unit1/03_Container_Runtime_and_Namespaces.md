# 3. Container Runtime & Process Isolation (Namespaces)

## What is a Container Runtime?

A **Container Runtime** is the software responsible for **creating, starting, stopping, and managing containers** on a host system.

> Think of it as the **execution engine** that turns a container image into a running container.

**Simple analogy:**
- Container Image → **Packed lunch box** (read-only, just the food)
- Container Runtime → **Person who opens it, serves food, and cleans up**

### Why is it needed?
A container image is just a **read-only package** (app + dependencies). To actually run it, the runtime:
1. Creates an isolated environment
2. Applies resource limits
3. Starts the application process
4. Monitors and stops it when required

---

## Types of Container Runtimes

### 1. High-Level Runtimes (used by developers/DevOps tools)
These handle image pulling, networking, storage, and container lifecycle.

| Runtime | Description |
|---------|-------------|
| **Docker Engine** | Most popular, full-featured |
| **containerd** | Industry standard, used by Kubernetes |
| **CRI-O** | Kubernetes-focused |
| **Podman** | Daemonless alternative to Docker |

> Docker provides: image management APIs, management interfaces, calls low-level runtime

### 2. Low-Level Runtimes (the actual container runner)
These directly interact with the Linux kernel to run container processes.

| Runtime | Description |
|---------|-------------|
| **runc** | OCI-compliant, most widely used |

They work with:
- **Linux kernel**
- **Namespaces**
- **cgroups**

---

## Process Isolation using Namespaces

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

> **Remember:** Namespaces = **Isolation**. cgroups = **Resource control**. (Next topic)

---

> 📅 *Notes from Class — Unit I*  
> 📖 *Reference: Slides (Slides 18–22) + Handwritten notes (Pages 3–5)*
