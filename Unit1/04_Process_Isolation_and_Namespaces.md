# Process Isolation & Namespaces

**What is Process Isolation?**

Process isolation = kernelfeature that separates processes so they can't interfere with each other.

Without isolation: Multiple applications running. Any app can access memory/files of other apps. One crashed app can crash entire system. Security nightmare.

With isolation: Each app runs in sandbox. App A can't see or touch app B's data/files/network. Crash isolated. Secure.

Docker uses Linux namespaces for process isolation.

Linux Namespaces:

Namespace = partition of system resources. Each namespace has isolated view of that resource.

Concept: Instead of one global resource, create separate "views" for each container. Container thinks it owns the resource.

Example: PID namespace
Global (without namespace): 1000s of process IDs on system (1, 2, 3, 1000, 1001, etc)
With PID namespace: Container A sees its processes as 1, 2, 3 (isolated view)
Container B sees its processes as 1, 2, 3 (different processes, isolated view)
Host sees all processes with actual IDs

Result: Containers think they're the only running apps, but they're isolated from each other.

Six Types of Namespaces:

1. PID Namespace (Process ID Isolation)

Purpose: Isolate process IDs and process tree.

How it works:
Each container gets own PID namespace.
Container's main application gets PID 1.
Other processes inside get PID 2, 3, 4, etc.

Example:
Container A running Nginx:
- Inside container: PID 1 (nginx process)
- On host: PID 5847 (same process, different view)

Container B running Node.js:
- Inside container: PID 1 (node process)
- On host: PID 6123 (same process, different view)

Why it matters:
- Processes isolated from each other
- Container thinks it's only running app
- Clean abstraction
- Host scheduler manages all processes fairly

Verification:
Inside container:
ps aux (shows only container's processes)
echo $$ (shows PID, usually 1)

On host:
docker exec container ps aux (same processes, different PIDs)

2. Network Namespace

Purpose: Isolate network interfaces, IP addresses, ports, routing.

How it works:
Each container gets own network interface.
Own IP address (172.17.0.2, 172.17.0.3, etc).
Own routing table.
Own port space.

Example:
Container A (Nginx):
- IP: 172.17.0.2
- Port 80 → listening
- Routes traffic via eth0 → 172.17.0.1 (gateway)

Container B (Node.js):
- IP: 172.17.0.3
- Port 80 → listening (NO CONFLICT because different IPs)
- Routes traffic via eth0 (different interface) → 172.17.0.1

Why it matters:
- Both containers can use port 80 (no conflict)
- Isolated network communication
- Network independently configurable
- Can have multiple IPs per container

Verification:
Inside container:
ip addr show (shows container's IP)
netstat -tlnp (shows listening ports)
ip route (shows routing table)

Connection example:
Container A reaches Container B:
ping 172.17.0.3 (works if on same Docker network)

3. Mount Namespace (Filesystem Isolation)

Purpose: Isolate filesystem structure and mounted volumes.

How it works:
Each container has own root filesystem (/).
Changes to filesystem visible only to that container.
Mount points isolated.

Example:
Container A (Ubuntu):
- Has its own /bin, /lib, /app directories
- File edit: /app/data.txt
- Visible only in Container A

Container B (Ubuntu):
- Has separate /bin, /lib, /app directories (copied from image)
- File edit: /app/data.txt (different file!)
- NOT visible in Container A

Why it matters:
- Containers have independent filesystems
- No file conflicts
- Each container has clean OS files
- Volumes can be mounted per container

Verification:
Inside container:
ls / (shows root filesystem)
mount (shows mounted volumes)
df -h (shows filesystem usage)

File isolation:
Container A: echo "data-A" > /data.txt
Container B: echo "data-B" > /data.txt
(Two different files, isolated)

4. IPC Namespace (Inter-Process Communication Isolation)

Purpose: Isolate IPC mechanisms (message queues, shared memory, semaphores).

How it works:
Each container has own IPC namespace.
Containers can't share IPC mechanisms.
Only processes within same IPC namespace can communicate.

Example:
Container A creates shared memory segment.
Container B can't access it (different IPC namespace).
Processes inside Container A can use it (same namespace).

Why it matters:
- Isolates inter-process communication
- Prevents unauthorized process communication across containers
- Each container's internal processes isolated together

IPC resources:
- Message queues (for passing messages between processes)
- Shared memory (processes share memory region)
- Semaphores (synchronization primitives)

Verification:
Inside container:
ipcs (shows IPC resources)
ipcs -m (shows shared memory)
ipcs -q (shows message queues)
ipcs -s (shows semaphores)

5. UTS Namespace (Hostname Isolation)

Purpose: Isolate hostname and NIS domain name.

How it works:
Each container can have own hostname.
Changes to hostname visible only in that container.
Host machine's hostname unchanged.

Example:
Container A:
- Hostname command: backend-server
- Inside: hostname → backend-server

Container B:
- Hostname command: frontend-server
- Inside: hostname → frontend-server

Host machine:
- Actual hostname: production-host
- Unchanged

Why it matters:
- Containers can have meaningful hostnames
- Useful for service identification
- Applications inside can identify container by name
- Independent configuration

Verification:
Inside container:
hostname (shows container's hostname)
hostname -f (shows fully qualified domain name)

Set hostname at runtime:
docker run -h mycontainer image (sets hostname to mycontainer)

6. User Namespace (User/Permission Isolation)

Purpose: Isolate user and group IDs, security context.

How it works:
Container's user ID mapped to host user ID.
Container root (UID 0) ≠ host root (UID 0).
Security boundary: container can't escape to get real root.

Example:
Inside Container:
- Running as UID 0 (appears as root)
- Actually mapped to UID 100001 on host (regular unprivileged user)

Security benefit:
- Container process runs with limited privileges
- Even if containerescapes, only gets unprivileged user access
- Can't gain actual root on host

Why it matters:
- Security isolation
- Prevents privilege escalation
- Protects host from compromised containers

Verification:
Inside container:
id (shows UID/GID)
whoami (shows username)

On host (with appropriate context):
Can see actual UID mapping

Process Isolation Mechanisms:

Namespace combined effects:

Single PID namespace:
- Container thinks only its processes exist
- Doesn't know about other containers/host processes

PID + Network namespace:
- Isolated processes + isolated network
- Container + processes + own IP/ports

PID + Network + Mount namespace:
- Isolated processes + isolated network + isolated filesystem
- Complete separation

PID + Network + Mount + IPC + UTS + User namespace:
- Complete isolation (what Docker typically sets up)
- Container is complete sandbox

Example Container Isolation:

Docker runs container with all 6 namespaces:

docker run -it ubuntu bash

Internally:
- Creates PID namespace (process isolation)
- Creates Network namespace (network isolation)
- Creates Mount namespace (filesystem isolation)
- Creates IPC namespace (IPC isolation)
- Creates UTS namespace (hostname isolation)
- Creates User namespace (user isolation)
- Container process assigned to all 6 namespaces

Result: Container is completely isolated sandbox.

How Containers See the System:

With all namespaces active:

ps aux:
PID 1 (bash)
PID 2 (any child processes)
(Only container's processes visible)

hostname:
(Custom hostname if set)

ip addr:
(Container's IP address)

ls /:
(Container's root filesystem)

id:
(Container user/group mappings)

df -h:
(Container's mount points, volumes)

Security Through Namespaces:

Isolation = Security

Scenario: Malicious container tries to attack:
1. Compromise Container A process
2. Escape PID namespace? Can't. Kernel enforces isolation.
3. Try to modify host filesystem? Can't. Mount namespace blocks it.
4. Try to access Container B network? Can't. Network namespace blocks it.
5. Try to access Container B memory? Can't. IPC namespace blocks it.
6. Try to gain root? Can't. Already mapped to unprivileged user.

Result: Container compromise isolated. Host/other containers safe.

Shared Kernel:

Important: Containers share host kernel!

Example:
Host runs Linux kernel 5.10
Container A: shares kernel 5.10
Container B: shares kernel 5.10
Container C: shares kernel 5.10

Benefit: One kernel for all → memory efficient, fast startup

Note: Can't run Windows containers on Linux kernel (kernel mismatch).

Namespace Limitations:

What namespaces DON'T isolate:
- CPU time (use cgroups for limits)
- Memory (use cgroups for limits)
- Disk I/O (use cgroups for limits)
- Host system calls (kernel still shared)

Solution: Use cgroups + namespaces together
- Namespaces = isolation (what you see)
- cgroups = resource control (what you get)

Namespaces + Cgroups Together:

Process A:
- PID namespace: sees PID 1
- cgroup CPU limit: max 1 core
- cgroup memory limit: max 512MB

Result:
- Isolated from other processes (namespace)
- Fair CPU/memory share (cgroups)

Complete Isolation = Namespaces + Cgroups

Verifying Namespaces:

List namespaces on system:
ip netns list (network namespaces)
lsns (all namespaces)

Inspect container namespaces:
docker inspect container_id (shows namespace URLs/info)

Example output:
"Pid": "2478" (PID namespace ID)
"NetworkMode": "bridge" (Network namespace type)
"IPAddress": "172.17.0.2" (container's IP)

Creating Containers with Custom Namespace Settings:

Shared network namespace (multi-container):
docker run -d --name server nginx
docker run --network container:server --name app ubuntu

server and app share network namespace!

No network (none namespace):
docker run --network none --name isolated ubuntu
(No network access, complete network isolation)

Host network (no network namespace):
docker run --network host --name hostnet nginx
(Uses host network directly, no isolation)

Practical Examples:

Example 1: PID Isolation Verification

docker run -d --name app1 ubuntu sleep 1000
docker exec app1 ps aux
(Shows only sleep process)

docker ps (on host)
(Shows container as one process, with different PID)

Example 2: Network Isolation

docker run -d -p 8080:80 --name web1 nginx
docker run -d -p 8081:80 --name web2 nginx

Both listening on port 80 (inside containers) but:
web1 accessible on localhost:8080
web2 accessible on localhost:8081
(No port conflict because network namespaces)

Example 3: Filesystem Isolation

docker run -v ~/data:/app/data --name app1 ubuntu bash
Inside: touch /app/data/file1.txt

docker run -v ~/data:/app/data --name app2 ubuntu bash
Inside: ls /app/data
(Sees file1.txt because same volume, but filesystem isolated)

Key Points Summary:

1. Namespaces isolate system resources (what container sees)
2. Six namespace types: PID, Network, Mount, IPC, UTS, User
3. Complete isolation = all 6 namespaces + cgroups
4. Containers share kernel (efficient)
5. Shared kernel restricts: can't mix OS types (no Windows on Linux)
6. Isolation provides security (malicious container contained)
7. Namespace limitations: only isolation, not resource control
8. cgroups needed for: CPU, memory, I/O limits
9. Namespaces + cgroups = perfect container foundation