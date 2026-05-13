# Topic 4: Containers

**[← Back to Unit 3](README.md)**

## Overview

Containers are OS-level virtualization technologies that enable microservices by providing lightweight, isolated execution environments.

---

## Definition

**Containers:**
> Containers are OS-level virtualization technologies that allow multiple isolated applications to run on a single host/instance. Each container packages an application with all its dependencies (code, libraries, runtime) in a lightweight, portable unit.

**Key Characteristics:**
- Lightweight (10-50MB each)
- Fast to start (milliseconds)
- Isolated execution (separate process)
- Portable (works on any OS with container runtime)
- Efficient (share host OS kernel)

---

## How Containers Work

### Container vs Virtual Machine

#### Virtual Machines (VMs)

```
┌────────────────────────────────────────┐
│          Physical Server               │
├────────────────────────────────────────┤
│      Hypervisor (KVM/VMware)           │
├──────────────────┬──────────────────┬──┤
│                  │                  │  │
│ ┌──────────────┐ │┌──────────────┐ │  │
│ │  VM 1        │ ││  VM 2        │ │  │
│ │ ┌──────────┐ │ ││┌──────────┐  │ │  │
│ │ │  OS      │ │ │││ OS       │  │ │  │
│ │ │ (Linux)  │ │ │││ (Linux)  │  │ │  │
│ │ └──────────┘ │ ││└──────────┘  │ │  │
│ │  App 1       │ ││ App 2        │ │  │
│ │  Libraries   │ ││ Libraries    │ │  │
│ └──────────────┘ │└──────────────┘ │  │
└──────────────────┴──────────────────┴──┘
```

**VM Characteristics:**
- **Full OS:** Each VM runs complete OS (500MB - 2GB)
- **Hypervisor:** Needed to manage VMs
- **Isolation:** Complete OS-level isolation
- **Startup:** Minutes to boot
- **Performance:** ~80-90% of native (hypervisor overhead)
- **Density:** 10-20 VMs per host
- **Use Case:** When complete OS isolation needed

#### Containers

```
┌────────────────────────────────────────┐
│          Physical Server               │
├────────────────────────────────────────┤
│    Host OS Kernel (Shared)             │
├──────────────┬──────────────┬──────────┤
│              │              │          │
│ ┌──────────┐ │┌──────────┐ │┌───────┐ │
│ │Container │ ││Container │ ││Contain│ │
│ │Runtime   │ ││Runtime   │ ││ Runtime  │
│ │App 1     │ ││App 2     │ ││App 3  │ │
│ │Libraries │ ││Libraries │ ││Lib    │ │
│ └──────────┘ │└──────────┘ │└───────┘ │
└──────────────┴──────────────┴──────────┘
```

**Container Characteristics:**
- **Shared Kernel:** All containers share host OS kernel
- **No Hypervisor:** Runs natively on OS
- **Process Isolation:** Isolation via namespaces & cgroups
- **Size:** 10-50MB per container
- **Startup:** Milliseconds
- **Performance:** ~99% of native (minimal overhead)
- **Density:** 100s-1000s of containers per host
- **Use Case:** Lightweight app isolation, microservices

---

## Comparison Table: VMs vs Containers

| Aspect | Virtual Machine | Container |
|---|---|---|
| **Isolation Level** | Full OS isolation | Process-level isolation |
| **Size** | 500MB - 2GB | 10-50MB |
| **Startup Time** | 3-5 minutes | 100-500 milliseconds |
| **Performance** | ~80-90% of native | ~99% of native |
| **Density** | 10-20 per host | 100s-1000s per host |
| **Resource Use** | High (OS overhead) | Low (kernel overhead) |
| **OS Support** | Any OS per VM | OS kernel must match |
| **Technology** | Hypervisor-based | Kernel-based |
| **When to Use** | Full isolation, multiple OSes | Lightweight, many apps |
| **Examples** | VMware, KVM, Hyper-V | Docker, podman, LXC |

---

## How Containers Achieve Isolation

### Namespaces (Isolation)

Namespaces separate system resources so each container has its own view:

| Namespace | What's Isolated | Container Sees |
|---|---|---|
| **PID** | Process IDs | Starts at PID 1 (own init) |
| **Network** | Network interfaces, ports | Own network stack, 127.0.0.1 |
| **Mount** | Filesystems | Own /root filesystem |
| **IPC** | Inter-process communication | Isolated message queues |
| **UTS** | Hostname | Own hostname |
| **User** | User IDs | Own user space |

**Example:**
```bash
# Host system
ps aux | grep nginx    # PID 1234
hostname               # server.example.com

# Inside container
ps aux | grep nginx    # PID 1 (different namespace)
hostname               # container_id (different namespace)

# Same process, different PID view!
```

### Control Groups (cgroups) — Resource Limits

cgroups limit resource consumption per container:

```yaml
# Container resources limited
memory: 512MB          # Max memory
cpus: 1.0             # Max CPUs
disk-read: 10MB/s     # I/O limits
disk-write: 10MB/s    # I/O limits
```

**Example:**
```bash
# Container with limits
docker run \
  --memory 512m \      # Max 512MB RAM
  --cpus 1.0 \         # Max 1 CPU
  --memory-swap 1g \   # Max swap 1GB
  nginx

# App tries to use 1GB RAM:
# Linux automatically kills process
```

---

## Benefits of Containers

### 1. Agile Application Deployment

```
Traditional Approach:
├── Develop on machine A
├── "Works on my machine"
├── Deploy to server B
├── "Doesn't work on production"
└── Debug environment differences

Containers:
├── Package with dependencies
├── Same container everywhere
├── "Works on my machine" → "Works everywhere"
└── No environment surprises
```

**Benefits:**
- Consistent environments
- No "it works on my laptop" problems
- Reliable deployments

### 2. Cost Efficiency

```
VMs Approach (Traditional):
├── 1000 VMs on 100 physical servers
├── Each VM: 500MB OS overhead
├── Total: 50GB OS overhead
└── Cost: High

Containers Approach:
├── 1000 containers on 10 physical servers
├── Each container: ~10MB overhead
├── Total: ~10GB overhead
└── Cost: 1/10th (same compute capacity)
```

**Benefits:**
- Better resource utilization
- More apps per server
- Lower cloud spending

### 3. Automated Deployment

```
# Simple command to deploy app
docker run my-app:latest

# Replaces hours of:
- OS installation
- Package installation
- Application configuration
- Permission setup
```

**Benefits:**
- Infrastructure as Code
- Consistent environments
- Easy scaling

### 4. Greater Isolation

```
Container 1: Memory leak
├── Limited by cgroup
├── Container 1 crashes
└── Container 2: Still running

vs

Monolithic VM:
├── Memory leak crashes entire app
└── All services down
```

**Benefits:**
- Resource-limited isolation
- One app doesn't affect others
- Better security boundaries

### 5. Better Resource Utilization

```
VMs (Traditional):
├── Web Server VM: 2GB allocated, using 200MB
├── Database VM: 4GB allocated, using 1GB
├── Cache VM: 2GB allocated, using 100MB
└── Total: 8GB allocated, 1.3GB used (very wasteful)

Containers (Modern):
├── Web: 256MB limit, using 256MB
├── Database: 2GB limit, using 1.5GB
├── Cache: 256MB limit, using 256MB
└── Total: 2.8GB (very efficient)
```

**Benefits:**
- Pack more apps per server
- Lower resource costs
- Better efficiency

### 6. Portability

```
"Build once, run anywhere"

├── Docker image built on laptop
├── Works on staging server
├── Works on production
├── Works on colleague's machine
├── Works on cloud (AWS, Azure, GCP)
└── Same container everywhere
```

**Benefits:**
- Works across environments
- Easy migration
- Cloud agnostic

### 7. Quick Response to Scaling

```
Traditional Scaling:
├── Get alert: traffic increased
├── Buy servers (takes weeks)
├── Install OS
├── Deploy application
└── Handle increased load (after 4 weeks)

Container Scaling:
├── Get alert: traffic increased
├── Start 10 more containers
└── Handle increased load (within seconds)
```

**Benefits:**
- Instant scaling
- Auto-scaling based on demand
- Cost savings (only run what needed)

---

## Containers + Microservices Synergy

Containers are **the perfect match** for microservices:

### Architecture Pattern

```
Kubernetes Cluster (Container Orchestration)
├── Node 1
│   ├── Container: User Service
│   ├── Container: Product Service
│   └── Container: Order Service
├── Node 2
│   ├── Container: Payment Service
│   ├── Container: Notification Service
│   └── Container: Cache Service
└── Node 3
    ├── Container: Search Service
    └── Container: Analytics Service
```

### Benefits of Containers for Microservices

| Benefit | Impact |
|---|---|
| **One service per container** | No resource contention between services |
| **Independent lifecycle** | Deploy, restart, scale one service |
| **Environment consistency** | Same image from dev → staging → production |
| **Resource limits** | CPU/memory enforced per service |
| **Easy orchestration** | Kubernetes manages containers at scale |
| **Density** | Run 100s of microservices on single cluster |
| **Auto-scaling** | Scale individual services independently |
| **Self-healing** | Automatically restart failed containers |

### Resilience Example

**Without Containers (Monolithic VM):**
```
Node crashes
├── VM terminated
├── Entire application down
├── All microservices offline
└── Complete service outage
```

**With Containers (Microservices):**
```
Node crashes
├── Containers on node terminated
├── Kubernetes detects failure
├── Reschedules containers to other nodes
├── User Service: ✓ Running (on Node 2)
├── Order Service: ✓ Running (on Node 3)
├── Payment Service: ✓ Running (on Node 2)
└── Application continues (with reduced capacity)
```

### Container Orchestration (Kubernetes)

When running 100s of containers:

```
Manual Approach:
├── Start container
├── Monitor resource usage
├── Restart if failed
├── Manually scale
├── Update running containers
└── Nightmare to manage

Kubernetes Approach:
├── Define desired state (10 Payment Service containers)
├── Kubernetes ensures it (restarts if any crashes)
├── Auto-scales based on CPU
├── Automatic rolling updates
├── Health checks and self-healing
└── Automated operations
```

---

## Container Technology Stack

### Container Runtimes

| Runtime | Description |
|---|---|
| **Docker** | Industry standard, most popular |
| **Podman** | Rootless containers, docker-compatible |
| **Containerd** | Lightweight, used by Kubernetes |
| **LXC** | Linux containers, lower-level |

### Image Registries (Where Images Stored)

| Registry | Purpose |
|---|---|
| **Docker Hub** | Public image registry (official images) |
| **Docker Registry** | Private on-premises registry |
| **Amazon ECR** | AWS private registry |
| **Google GCR** | Google Cloud private registry |
| **Azure ACR** | Microsoft Azure private registry |

### Orchestration Platforms (Manage Containers at Scale)

| Platform | Use Case |
|---|---|
| **Docker Compose** | Single machine, development |
| **Docker Swarm** | Multiple machines (deprecated) |
| **Kubernetes** | Production, enterprise-scale |
| **Nomad** | Multi-platform orchestration |

---

## Container Best Practices

### 1. One Process Per Container

```yaml
# ✓ Good
web:
  image: nginx
  # Just runs nginx

# ✗ Bad
web:
  image: myapp
  # Runs nginx + app + ssh + monitoring
```

### 2. Immutable Infrastructure

```dockerfile
# ✓ Good - Create new image for each version
FROM ubuntu:22.04
RUN apt-get update && apt-get install nginx
COPY app /var/www/
CMD ["nginx"]

# ✗ Bad - Modify running container
docker exec container apt-get install package
```

### 3. Minimal Base Images

```dockerfile
# ✓ Good (500MB)
FROM ubuntu:22.04
RUN apt-get install app

# ✗ Bad - Large base (1GB+)
FROM ubuntu:22.04
RUN apt-get install app
RUN apt-get install build-essentials
RUN apt-get install dev-tools
```

### 4. Security

```dockerfile
# ✓ Good - Run as non-root
USER appuser
CMD ["app"]

# ✗ Bad - Run as root (security risk)
CMD ["app"]
```

---

## Key Takeaway

**Containers provide lightweight, portable, scalable environments that enable microservices architecture.**

```
Containers = Optimal foundation for microservices

Why?
├── Lightweight (not resource-heavy like VMs)
├── Fast (milliseconds startup)
├── Portable (works everywhere)
├── Scalable (100s-1000s per host)
├── Isolated (services don't interfere)
└── Efficient (share OS kernel)
```

---

## Next Topic

**[Read about Docker Compose →](05_Docker_Compose.md)**

Learn how Docker Compose simplifies running multiple containers together.
