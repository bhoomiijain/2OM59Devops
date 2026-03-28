# Images & Layers

## Image = blueprint/template

Read-only = doesn't change once made

Contains:
- OS (small)
- App code
- Libraries
- Runtime
- Config

**Example:** `python:3.11-slim`  
Has Linux + Python 3.11 already

When you run it → Docker makes container (adds writable top layer)

## Layers

Image = stack of layers

```
Layer 4: App
Layer 3: Libraries  
Layer 2: Python
Layer 1: Linux
```

**Each Dockerfile line = 1 layer**

**Why layers matter:**
- Immutable = once made, don't change
- Cached = Docker reuses them (faster building)
- Shared = same layer used in multiple images (saves space)

**Good:** Faster builds, less storage, quick deployments

## Copy-on-Write??

When container edits file from lower layer = file copied to writable layer

Original stays unchanged

---


