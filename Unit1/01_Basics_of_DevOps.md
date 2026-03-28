# 1. Basics of DevOps & Why Docker

## What is DevOps?
DevOps is a combination of **Development** and **Operations**. It bridges the gap between writing code and deploying/running it in production.

**Key goals:**
- Faster software delivery
- Better collaboration between dev and ops teams
- Automation of repetitive tasks (build, test, deploy)

---

## Why DevOps Needs Containers

### The old problem:
- Applications were **monolithic** → ran on single physical servers
- As apps grew → split into components (frontend, backend, database)
- VMs handled hosting, but they were **heavy and slow**
- Now apps are **microservices** → containers are the solution

### What Containers Solve:
| Problem | Container Solution |
|--------|-------------------|
| "Works on my machine" | Same image runs everywhere |
| Slow VM startup | Containers start in seconds |
| High resource usage | Containers share host OS kernel |
| Portability issues | Build once, run anywhere |

---

## Course Syllabus — INT332

| Unit | Topic |
|------|-------|
| I | Intro to Containers (Origin, DevOps basics, infrastructure) |
| II | Image Building & Container Management, Dockerfile |
| III | Microservices with Docker Compose |
| IV | Maven Build Automation |
| V | CI (Continuous Integration) with GitHub Actions |
| VI | CI/CD with Jenkins |

**Assessment:**
- AT1: BYOD
- AT2: Project
- Marks split: 5, 45, 50
- Project: 30 marks → Presentation, execution, tools, building CI/CD, synopsis, viva
- Git repo: 30 marks → Complete git maintenance

**Tools needed:**
- Docker Desktop / AWS Platform
- DockerHub (global sharing & versioning of images — free)
- Jenkins for CI/CD
- Maven

---

## Key Terminology

| Term | Meaning |
|------|---------|
| **Container** | Lightweight isolated environment to run an app |
| **Image** | Blueprint/template to create containers |
| **Docker** | Most popular containerization platform |
| **DevOps** | Culture + tools for fast, reliable software delivery |
| **CI/CD** | Continuous Integration / Continuous Deployment |

---


