# 5. Container Images & Layers

## What is a Container Image?

A **Container Image** is a **read-only blueprint** for creating containers.

> Container Image = **Instructions** (blueprint)  
> Container = **Writable layer** + executable file (the running instance)

**A container image contains:**
- Base OS (minimal)
- Application code
- Libraries & Dependencies
- Runtime environment
- Configuration

**Important:** Images are **immutable** — once built, they do not change.

---

## Real Example

`python:3.11-slim` is an image that includes:
- Minimal Linux OS
- Python 3.11
- Standard Python libraries

When you run it → Docker creates a **container** from this image (adds a writable layer on top).

---

## Image Layers

Each image is built as a **stack of layers**.

```
Layer 4: [ Application Layer ]
Layer 3: [ Dependencies Layer ]
Layer 2: [ Runtime Layer (Python 3.11) ]
Layer 1: [ Base OS Layer (Linux) ]
```

**Each instruction in a Dockerfile = one layer**

### Properties of Layers:
- **Immutable** — layers never change once created
- **Cached** — Docker reuses unchanged layers (faster builds)
- **Shared** — same layer can be reused across multiple images (saves storage)

### Benefits of Layered Architecture:

| Benefit | How |
|---------|-----|
| Faster Builds | Unchanged layers are reused from cache |
| Smaller Storage | Same layers shared between images |
| Faster Deployment | Only missing/new layers are downloaded |

---

## Image vs Container

| Concept | Analogy | Technical meaning |
|---------|---------|------------------|
| Image | Recipe book | Read-only blueprint |
| Container | Cooked food | Running instance with writable layer |

---

## Docker Layering & Filesystem (Union Filesystem)

Docker uses a **Union Filesystem** (like OverlayFS) to:
- Stack layers on top of each other
- Present them as a **single unified filesystem** to the container
- When a container modifies a file → it's written to the **top writable layer** (Copy-on-Write mechanism)

**Copy-on-Write (CoW):** Files from lower (read-only) layers are **copied up** to the writable layer when modified. Original layers remain unchanged.

---

> 📅 *Notes from Class — Unit I*  
> 📖 *Reference: Slides (Slides 33–38) + Handwritten notes (Pages 6–7)*
