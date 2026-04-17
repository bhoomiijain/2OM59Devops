# Container Runtime

What is Container Runtime?

Container runtime = software that creates, starts, stops, and manages containers.
Takes Docker image → makes it run as a container.

Why needed?

Developers build images (complete application packages).
Someone needs to execute those images into running containers.
Container runtime does that job - the execution engine.

Examples of Container Runtimes:

Docker:
- Most popular, user-friendly
- High-level runtime
- Manages images, networking, storage
- Used by most developers

containerd:
- Lightweight high-level runtime
- Used by Kubernetes
- Focused on container lifecycle
- Smaller than Docker

CRI-O:
- Container Runtime Interface for Kubernetes
- Kubernetes-native runtime
- Alternative to containerd

Podman:
- Docker alternative
- Doesn't need daemon (rootless by default)
- Better for security-conscious environments

What Container Runtime Does:

1. Image Management
- Reads image from registry (Docker Hub, GHCR, etc)
- Validates image integrity
- Prepares image filesystem

2. Container Creation
- Creates container filesystem from image layers
- Sets up working directory
- Initializes environment

3. Networking Setup
- Creates network interfaces
- Assigns IP address
- Sets up DNS
- Configures port mapping

4. Resource Allocation
- Assigns CPU quota (cgroups)
- Allocates memory limit
- Sets disk I/O limits
- Configures network bandwidth limits

5. Process Isolation
- Creates namespaces (PID, Network, Mount, IPC, UTS, User)
- Container isolated from host/other containers
- Enforces security boundaries

6. Container Execution
- Starts main application process
- Runs with specified command (from Dockerfile or command line)
- Manages process lifecycle

7. Monitoring & Lifecycle
- Watches container for termination
- Handles restart policies
- Collects logs
- Manages resource monitoring

High-Level vs Low-Level Runtimes:

Low-Level Runtime (OCI Runtime):

Definition: Works directly with kernel features, creates actual container processes.

Examples:
- runc: Most common, part of Docker
- crun: Alternative implementation, faster startup  
- kata: Lightweight VMs instead of containers
- gVisor: Sandboxed runtime

Responsibilities:
- Reads OCI bundle (standardized container format)
- Creates namespaces
- Applies cgroups
- Mounts filesystems
- Executes container process
- Handles signals (SIGTERM, SIGKILL)

Users: Rarely used directly, called by high-level runtimes.

High-Level Runtime:

Definition: User-friendly interface on top of low-level runtime.

Examples:
- Docker: Image management + networking + storage + low-level runtime
- containerd: Container lifecycle + low-level runtime
- CRI-O: Kubernetes integration + low-level runtime
- Podman: Daemon-less + low-level runtime

Responsibilities:
- Image pulling/pushing/management
- Network management (create/delete networks)
- Volume management
- Easy-to-use CLI
- Security policies
- Logging aggregation

Users: Everyone! Developers use high-level runtimes daily.

Example Runtime Stack:

User runs:
docker run -p 8080:80 nginx

Flow:
1. Docker CLI (docker command)
2. Docker Daemon (dockerd) [high-level runtime]
3. containerd [high-level runtime for K8s]
4. runc [low-level runtime, OCI compliant]
5. Linux kernel [creates namespaces, applies cgroups]

Visual:
User → docker run
    ↓
Docker (high-level)
    ↓
containerd (high-level for K8s)
    ↓
runc (low-level)
    ↓
Linux kernel (namespaces, cgroups)
    ↓
Container process runs

Container Runtime Configuration:

Default runtimes:
Docker uses runc by default.
Kubernetes can use containerd or CRI-O.

Custom runtimes:
Can specify different runtime for different containers:
docker run --runtime=crun nginx (use crun instead of runc)
docker run --runtime=kata nginx (use kata for lightweight VMs)

Why different runtimes?

runc: Standard, stable, widespread.
crun: Faster startup time.
kata: More security (lightweight) VM isolation.
gVisor: Sandboxed, best security, slower.

Runtime Performance Comparison:

runc:
- Startup: ~100ms
- Memory overhead: minimal
- Security: good (namespace-based)

crun:
- Startup: ~50ms (faster)
- Memory overhead: minimal
- Security: good (namespace-based)

kata:
- Startup: ~1000ms (slower)
- Memory overhead: ~50MB (VM overhead)
- Security: excellent (VM isolation)

Runtime Standards:

OCI (Open Container Initiative):

Standard specification for:
- Image format
- Runtime specification
- Distribution specification

Ensures containers portable across runtimes:
- Image built for Docker ✓ runs on containerd ✓ runs on CRI-O

Runtime Lifecycle:

When container created:

1. Runtime receives
container creation request

2. Runtime reads OCI bundle:
specification.json (container config)
rootfs (filesystem)

3. Runtime setup:
Create namespaces
Create cgroups
Mount filesystems
Create container root filesystem

4. Runtime execute:
Execute entrypoint/command
Process runs as PID 1

5. Runtime monitor:
Watch for process termination
Handle signals
Collect exit code

6. Runtime cleanup:
Destroy namespaces
Release cgroups
Remove mount points
Clean up filesystem

Runtimes and Kubernetes:

Kubernetes needs container runtime.

CRI (Container Runtime Interface):
Standard interface for Kubernetes to talk to container runtimes.

Docker uses dockerd → Docker shim → containerd → runc.
Kubernetes directly uses containerd or CRI-O (more efficient).

Container Runtime Selection:

Development:
- Docker (familiar, full-featured)

Production:
- containerd (lightweight, efficient)
- CRI-O (Kubernetes-native)

High-security:
- kata (VM-based isolation)
- gVisor (sandboxed)

Best practices:

1. Use standard OCI-compliant runtime (portable)
2. Match runtime to use case (dev, production, security)
3. Keep runtime updated (performance, security)
4. Use appropriate runtime for workload (CPU vs security)
5. Monitor runtime performance (startup, memory, CPU)

