# 2. Virtualization vs Containerization

## What is Virtualization?

Virtualization is the process of **partitioning one physical server into multiple Virtual Machines (VMs)**.

**How it works:**
- A software called **Hypervisor** divides the physical server
- Each VM gets its own **OS, memory, and CPU**
- Each VM acts like a completely separate physical machine

```
[ Guest OS ]  [ Guest OS ]  [ Guest OS ]
[   VM 1   ]  [   VM 2   ]  [   VM 3   ]
         [ Hypervisor ]
              [ Host OS ]
      [ H/W: network, storage, compute ]
```

### Advantages of Virtualization:
- Strong isolation between VMs
- Can run different OS (Linux + Windows on same hardware)
- Better hardware utilization than single-server setup
- Better disaster recovery — infected VM can be isolated
- Saves space and hardware cost

### Drawbacks of Virtualization (Why we needed containers):
- **Heavy resource overhead** — each VM has full OS
- **Slow boot time** — minutes to start
- **Poor scalability** — spinning up many VMs is expensive
- **Limited portability** — VMs are bulky
- **High management complexity**

---

## What is Containerization?

Containers are a **lightweight alternative to VMs**.

**Key difference from VMs:**
> Containers don't carry an entire OS. They use **bare minimum OS**.

```
[ app ]  [ app ]  [ app ]          [ app ]  [ app ]  [ app ]
[libs]   [libs]   [libs]   VS      [libs]   [libs]   [libs]
[Guest OS] [Guest OS] [Guest OS]       [ Docker Engine ]
        [ Hypervisor ]                      [ O.S. ]
           [ Host OS ]                  [ Infrastructure ]
        [ Infrastructure ]
         ← VMs →                        ← Containers →
```

### How Containers Work:
- They **share the host OS kernel**
- Only bring what the **application needs**
- Start in **seconds**
- Consume **minimal resources**
- You can run hundreds on the same machine

### Real-World Analogy:
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

---

> 📅 *Notes from Class — Unit I*  
> 📖 *Reference: Slides + Handwritten notes (Pages 2–3)*
