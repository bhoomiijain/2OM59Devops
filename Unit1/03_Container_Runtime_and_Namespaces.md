# Container Runtime & Namespaces

Container Runtime:

Definition: Software that creates, starts, stops, and manages containers. Takes image → makes it run.

Why needed? Developers build images (apps). Someone needs to execute those images. Container runtime does that job.

Examples:
- Docker (most popular, easy to use)
- containerd (lighter, used by Kubernetes)
- CRI-O (alternative, built for Kubernetes)
- Podman (Docker alternative, rootless by default)

What container runtime does:
- Reads image from registry
- Creates container filesystem
- Sets up networking
- Allocates resources (CPU, memory, disk)
- Runs application process
- Manages lifecycle (start, stop, pause, remove)
- Isolates container using namespaces
- Limits resources using cgroups

High-level vs Low-level Container Runtimes:

Low-level runtime:
- Works directly with kernel features (namespaces, cgroups)
- Creates actual container processes
- Examples: runc, crun, kata
- Not directly used by users
- Called by high-level runtimes

High-level runtime:
- User-friendly interface (Docker, Podman)
- Image management, networking, storage
- Uses low-level runtime under the hood
- Examples: Docker, containerd, CRI-O, Podman

Example stack:
User runs "docker run" → Docker (high-level) → containerd (high-level for K8s) → runc (low-level) → Linux kernel features

Process Isolation using Namespaces:

Problem: Multiple containers running on same host. How do they not interfere with each other?

Solution: Linux namespaces. Partition system resources so each container sees only its own environment.

How? Kernel creates separate namespace for each container. Container thinks it owns:
- Own processes (PID namespace)
- Own network stack (Network namespace)
- Own filesystem (Mount namespace)
- Own users (User namespace)
- Own hostname (UTS namespace)
- Own IPC system (IPC namespace)

Result: Container isolated from host and other containers. But shares kernel (efficient).

Types of Namespaces:

1. PID Namespace (Process Isolation)
Each container has own process tree. Processes inside = isolated from host/other containers.
Inside container: PID 1 (app's main process)
Host sees: PID 5847 (same process)
Result: Container thinks it's the only app running. Host sees it as one process among many. Clean isolation.

2. Network Namespace
Each container gets own IP address, network interfaces, routing table.
Result: Two containers can both listen on port 8080 (no conflict).
Example:
Container A: port 8080 on 172.17.0.2
Container B: port 8080 on 172.17.0.3
No conflict. Same port, different IPs.

3. Mount Namespace (Filesystem/Volume Isolation)
Each container has own filesystem view.
Container A: sees /app/data
Container B: sees /app/data (different location, different data)
File changes inside container A don't affect container B or host.
Result: Containers have isolated storage.

4. IPC Namespace (Inter-Process Communication)
Each container has own message queues, shared memory segments.
Containers can't communicate between each other via IPC (isolated).
Result: Process communication isolated.

5. UTS Namespace (Hostname/Domain Isolation)
Each container can have own hostname.
Container A: hostname = "backend"
Container B: hostname = "frontend"
Result: Containers can have different hostnames.

6. User Namespace (User/Group Isolation)
Container user mapped to host user (usually non-root).
Container root ≠ host root (security).
Example: Container's root user = host's regular user (no actual root privileges).
Result: Better security (container can't escape and get real root).

Why namespaces matter?

Isolation = security. Each container isolated from others. Compromised container can't attack others.
Efficiency = shared kernel. Don't need separate OS per container (unlike VMs).
Cleanliness = containers think they own system, but don't. Creates clean abstraction.

Remember: namespaces = isolation (what each container sees), Cgroups = resource control (how much resource each container gets)

