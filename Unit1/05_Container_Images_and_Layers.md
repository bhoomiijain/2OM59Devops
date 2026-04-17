# Images & Layers

Image = blueprint/template. Read-only - doesn't change once made.

Contains: OS (small), app code, libraries, runtime, config.

Example: python:3.11-slim has Linux + Python 3.11 built in.

When you run it: Docker makes container and adds writable top layer.

Image = stack of layers

Layer 4: App
Layer 3: Libraries
Layer 2: Python
Layer 1: Linux

Each Dockerfile line = 1 layer

Why layers matter:
- Immutable: once made, don't change
- Cached: Docker reuses layers (faster builds)
- Shared: same layer in multiple images (saves space)
- Result: faster builds, less storage, quick deployments

Copy-on-Write: when container edits file from lower layer, file copied to writable layer. Original stays unchanged.


