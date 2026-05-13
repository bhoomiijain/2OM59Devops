# Unit 3: Microservices and Containers

Complete study notes on microservices architecture, containers, and Docker Compose.

## 📚 Topics

1. **[Evolution of Application Architecture](01_Evolution_of_Application_Architecture.md)**
   - Three phases of application deployment
   - From physical servers to cloud-native microservices

2. **[Monolithic Applications](02_Monolithic_Applications.md)**
   - Definition and structure
   - Core components (UI, DAL, Data Store)
   - Advantages and disadvantages

3. **[Microservices Architecture](03_Microservices_Architecture.md)**
   - Core concepts
   - Real-world examples (Amazon, Netflix)
   - Database per service pattern
   - Microservices vs Monolithic comparison

4. **[Containers](04_Containers.md)**
   - What are containers?
   - Containers vs Virtual Machines
   - Benefits of containers
   - Containers + Microservices synergy

5. **[Docker Compose](05_Docker_Compose.md)**
   - What is Docker Compose?
   - Why do we need it?
   - Design principles

6. **[Docker Compose — Building Blocks](06_Docker_Compose_Building_Blocks.md)**
   - YAML structure
   - Services, images, ports, volumes, networks
   - Environment variables and restart policies

7. **[Docker Compose — Essential Commands](07_Docker_Compose_Commands.md)**
   - Initialization & validation
   - Starting and stopping services
   - Monitoring and debugging
   - Complete command reference

8. **[Practical Examples](08_Practical_Examples.md)**
   - Example 1: Node.js + MongoDB
   - Example 2: WordPress + MySQL
   - Example 3: React + Spring Boot + PostgreSQL (3-Tier Application)

---

## 🎯 Quick Navigation

| Topic | Key Concepts |
|---|---|
| [Architecture Evolution](01_Evolution_of_Application_Architecture.md) | Physical servers → Virtualization → Cloud-native |
| [Monolithic Apps](02_Monolithic_Applications.md) | Single unit, tightly coupled, simple to start |
| [Microservices](03_Microservices_Architecture.md) | Decomposed, independent, scalable services |
| [Containers](04_Containers.md) | Lightweight virtualization, OS-level isolation |
| [Docker Compose](05_Docker_Compose.md) | Multi-container orchestration, YAML config |
| [Building Blocks](06_Docker_Compose_Building_Blocks.md) | Services, volumes, networks, environment |
| [Commands](07_Docker_Compose_Commands.md) | up, down, ps, logs, exec, build |
| [Examples](08_Practical_Examples.md) | Node+Mongo, WordPress+MySQL, 3-Tier App |

---

## 📖 Learning Path

**Start here:** [Evolution of Application Architecture](01_Evolution_of_Application_Architecture.md)

**Then understand:** [Monolithic Applications](02_Monolithic_Applications.md) vs [Microservices Architecture](03_Microservices_Architecture.md)

**Learn the technology:** [Containers](04_Containers.md) and [Docker Compose](05_Docker_Compose.md)

**Deep dive:** [Building Blocks](06_Docker_Compose_Building_Blocks.md) and [Commands](07_Docker_Compose_Commands.md)

**Practice:** [Practical Examples](08_Practical_Examples.md)

---

## ✅ Learning Outcomes

By completing this unit, you will understand:

✅ Evolution of application architecture from monolithic to microservices  
✅ Structure and limitations of monolithic applications  
✅ Microservices architecture principles and benefits  
✅ Container technology and its advantages  
✅ Docker Compose for multi-container orchestration  
✅ How to write and manage docker-compose.yml files  
✅ Essential Docker Compose commands  
✅ Building real-world applications with Docker Compose  

---

## 🚀 Quick Command Reference

```bash
# Start services
docker-compose up -d

# View status
docker-compose ps

# Stream logs
docker-compose logs -f

# Enter container
docker-compose exec <service> bash

# Stop services
docker-compose down
```

---

**Last Updated:** May 2024  
**Version:** 1.0
