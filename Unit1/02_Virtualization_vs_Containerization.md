# VMs vs Containers

## Virtualization (VMs)

Running multiple OSes on one machine using Hypervisor

**Setup:**
- Hypervisor divides server
- Each VM = full OS + memory + CPU
- Like separate computers on same hardware

**Good:**
- Strong isolation
- Can run Linux AND Windows together
- Better space usage
- Disaster recovery possible

**Bad:**
- HEAVY - full OS = big size
- Slow (takes minutes to start)
- Expensive resource-wise
- Too much overhead

## Containers (Better option)

Lightweight = shares host OS instead of carrying full OS

**How?**
- Shares kernel with host
- Only takes what app needs 
- Starts in seconds!
- Uses minimal resources
- 100s of containers on same machine

**KEY DIFF:** VMs = full OS each | Containers = shared kernel

---
*Note: Teacher mentioned this but went fast... need to revise diagrams*
| Concept | Analogy |
|---------|---------|
| VM | Full house with kitchen, bathroom, etc. |
| Container | Hotel room — just what you need |
| Facebook app | Full house |
| Facebook Lite | Hotel room (container) |

---

## VM vs Container — Quick Comparison

| Feature | Virtual Machine | Container |
|---------|----------------|-----------|
| OS | Each has its own Guest OS | Shares host OS kernel |
| Boot time | Minutes | Seconds |
| Size | GBs | MBs |
| Isolation | Strong (full OS) | Process-level |
| Portability | Low | High |
| Resource usage | High | Low |
| Use case | Running different OS | Microservices, DevOps |

---

## Why Containers Won

1. Apps evolved: **Monolithic → Components → Microservices**
2. VMs handled component-split apps well
3. But microservices need **fast, lightweight, portable** environments
4. **Containers = perfect fit for microservices**
5. Portability problem solved — same container runs on dev laptop and production server

