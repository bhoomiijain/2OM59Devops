# cgroups

What are cgroups?

cgroups (control groups) = Linux kernel feature that limits and controls resources for processes and containers.

Why needed?

Without cgroups: One container can hog all CPU, RAM, disk I/O. Other containers starve. Host crashes.

With cgroups: Each container limited to fair share. System stays stable.

Namespaces vs cgroups:

Namespaces = isolation (what each container SEES)
- Container A sees its PID as 1 (isolation)
- Container B sees its PID as 1 (isolation)
- Both running on same host but don't see each other

cgroups = resource control (what each container GETS)
- Container A limited to 512MB RAM
- Container B limited to 256MB RAM
- CPU time fairly divided

Analogy: Apartment building

Namespaces = separate apartments (each tenant sees own walls, doesn't see neighbors)
cgroups = utilities meter on each apartment (electrical meter limits power usage per apartment)

Without cgroups = BAD:

One container hogs all resources:
- Tenant A runs heavy workload (CPU maxed out)
- Tenant B's app slows to crawl (no CPU left)
- Tenant C's app crashes (no memory left)
- Host server freezes
- All services down

With cgroups = GOOD:

Resources fairly divided:
- Tenant A gets 40% CPU (still works)
- Tenant B gets 35% CPU (still works)
- Tenant C gets 25% CPU (still works)
- Each container gets guaranteed minimum resources
- Fair sharing even under heavy load

Types of resources cgroups control:

1. CPU
Limit maximum CPU core usage (single or multiple cores).
Limit CPU time percentage (e.g., 50% of available CPU).
Set CPU priorities (some containers more important than others).
Result: Container can't monopolize CPU.

2. Memory
Limit maximum RAM usage (e.g., 512MB per container).
Soft limit (warning when exceeded) vs hard limit (kill container if exceeded).
Result: Container can't exhaust host memory.

3. Disk I/O
Limit read/write operations per second.
Limit bandwidth (MB/s) for disk operations.
Result: Container can't saturate disk (affects other containers).

4. Network
Limit bandwidth per interface (download/upload speed).
Rate limiting (packets per second).
Result: Container can't flood network.

5. Device Access
Limit access to device nodes (/dev/sda, etc).
Block specific devices.
Result: Container can't directly access host hardware.

6. Process Count
Limit number of processes/threads per container.
Result: Container can't fork bomb the system.

How cgroups work:

Linux creates cgroup for each container.
Docker sets limits on that cgroup:
docker run -m 512m nginx = 512MB RAM limit
docker run --cpus 1 nginx = 1 CPU core limit
docker run -e MEMORY_LIMIT=256m nginx = 256MB limit

Kernel enforces limits.
If container exceeds limit:
- Soft limit: warning (slows down)
- Hard limit: killed (container stops)

Result: Fair resource sharing among all containers.

Summary:

cgroups = resource police
Prevents one container from monopolizing resources.
Ensures fair sharing among multiple containers.
System stays stable even under heavy load.

Without cgroups: chaos (containers fighting for resources)
With cgroups: order (fair sharing, predictable behavior)

