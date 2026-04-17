# Container Runtime & Namespaces

Container runtime = software that creates/starts/stops containers. Examples: Docker, containerd, CRI-O, Podman. Docker most used. Does: runs processes, isolates stuff, sets resource limits, manages lifecycle.

Namespaces = Linux kernel feature that isolates processes. Containers think they're separate from each other and host.

Types of namespaces:
- PID: process IDs (each container sees different PID 1)
- Network: separate network (own IP, ports)
- Mount (mnt): separate file systems
- IPC: inter-process communication
- UTS: hostname stuff
- User: different users in containers

Example: Container A thinks PID 1 is its app, but host sees it differently. Each container = isolated world.

PID Namespace: each container has own process tree. PIDs inside start from 1. Container cannot see host or other container processes.

Container view:    Host view:
PID 1 → app       PID 1547 → app
PID 2 → worker    PID 1548 → worker

Network Namespace: each container gets own IP and ports. Two containers can both use port 8080 - no conflict.

Mount Namespace: each container has own filesystem view. File changes inside don't affect host.

UTS Namespace: allows container to have own hostname.

User Namespace: maps container user to non-root user on host. Improves security.

Remember: namespaces = isolation, cgroups = resource control

