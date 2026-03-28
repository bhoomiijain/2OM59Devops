# 4. Control Groups (cgroups) for Resource Limits

## What are cgroups?

**Control Groups (cgroups)** are a **Linux kernel feature** that limits, controls, and monitors resource usage of processes.

> **Namespaces = Isolate** | **cgroups = Control**

Without cgroups → one container could consume ALL CPU or RAM → host and other containers crash.

---

## Why cgroups are Critical

**Real-world analogy:**  
Think of an apartment building with an **electricity meter for each flat**.  
Each flat has a limit — one tenant cannot use all the power in the building.

Similarly, cgroups ensure:
- **No container can monopolize system resources**
- Every container gets a **fair share**

---

## Types of Resources Controlled by cgroups

| Resource | What cgroups does |
|----------|------------------|
| **CPU** | Limits CPU usage, assigns CPU shares |
| **Memory (RAM)** | Sets max memory; kills container if limit exceeded |
| **Disk I/O** | Limits read/write speeds |
| **Network** | Controls traffic (indirectly, through traffic shaping) |

---

## What happens WITHOUT cgroups?

- One container can consume **all CPU or RAM**
- Host system becomes **unresponsive**
- Other containers **crash**

**With cgroups:**
- Each container is **capped** at its resource limit
- System stays **stable** even under heavy load

---

## cgroups vs Namespaces — Quick Recap

| Feature | Namespaces | cgroups |
|---------|-----------|---------|
| Purpose | Isolation — hides resources | Limits — caps usage |
| What it does | Each container sees its own view | Controls how much resource each can use |
| Example | Container has PID 1 (isolated) | Container gets max 512MB RAM |

---

