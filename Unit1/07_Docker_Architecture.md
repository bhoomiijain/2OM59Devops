# Docker Architecture & Lifecycle

## Main Components

**Docker Client** = CLI (terminal where you type)  
**Docker Daemon** = engine (does the work)  
**Images** = read-only blueprint  
**Containers** = running instance  
**Docker Hub** = storage/registry  

## How Docker Works

You type command → CLI sends to Daemon → Daemon does stuff → result

**Like ordering food:**
- You = Client (customer)
- Chef = Daemon (does the cooking)
- Recipe = Image (instructions)
- Food = Container (actual result)

## Docker Daemon (dockerd)

Background service that:
- Listens for commands
- Creates/stops/manages containers
- Handles images
- Network stuff
- File storage

## Docker CLI

Commands you type:
- `docker run image` = create + start container
- `docker ps` = list running containers
- `docker stop <id>` = stop container
- `docker build` = make image
- `docker push` = upload image
- `docker pull` = download image

## Container Lifecycle

```
Created → Running → Paused → Stopped → Removed
```

1. **Create** - `docker create image` (ready but not running)
2. **Start** - `docker start <id>` (now running)
3. **Run** - `docker run` (create + start together)
4. **Stop** - `docker stop <id>` (pause execution)
5. **Remove** - `docker rm <id>` (delete container)

## Build, Ship, Run

**BUILD:** Make image with `docker build`
**SHIP:** Push to hub with `docker push`  
**RUN:** Start using `docker run`

## Docker vs VMs

| | Docker | VM |
|--|--------|-----|
| OS | Shares kernel | Separate OS |
| Startup | Seconds | Minutes |
| Size | Small | Large |
| Resources | Low | High |

Docker faster + lighter = better

---
*Note: Confused about docker images vs containers - ask for more examples*
