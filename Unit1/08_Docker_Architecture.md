# Docker Architecture & Lifecycle

Main Components:
- Docker Client = CLI (terminal commands)
- Docker Daemon = engine (does the work)
- Images = read-only blueprint
- Containers = running instance
- Docker Hub = storage/registry

How Docker works: Type command → CLI sends to Daemon → Daemon does stuff → result

Like ordering food: You = client (customer), Chef = daemon (cooking), Recipe = image (instructions), Food = container (result)

Docker Daemon (dockerd) = background service:
- Listens for commands
- Creates/stops/manages containers
- Handles images
- Network stuff
- File storage

Docker CLI commands:
- docker run image = create + start container
- docker ps = list running containers
- docker stop <id> = stop container
- docker build = make image
- docker push = upload image
- docker pull = download image

Container Lifecycle:
Created → Running → Paused → Stopped → Removed

1. Create: docker create image (ready but not running)
2. Start: docker start <id> (now running)
3. Run: docker run (create + start together)
4. Stop: docker stop <id> (pause execution)
5. Remove: docker rm <id> (delete)

Build, Ship, Run:
- BUILD: make image docker build
- SHIP: push to hub docker push
- RUN: start docker run

Docker vs VMs:
- OS: Docker shares kernel, VMs separate
- Startup: Docker seconds, VMs minutes
- Size: Docker small, VMs large
- Resources: Docker low, VMs high
