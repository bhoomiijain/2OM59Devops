# Topic 7: Docker Compose — Essential Commands

**[← Back to Unit 3](README.md)**

## Overview

Master the essential Docker Compose commands for managing multi-container applications.

---

## Command Syntax

**Basic Format:**
```bash
docker-compose [OPTIONS] COMMAND

# Modern syntax (Docker 20.10+)
docker compose [OPTIONS] COMMAND

# Old syntax (still works)
docker-compose [OPTIONS] COMMAND
```

> **Note:** Both `docker compose` and `docker-compose` work identically. Docker v20.10+ recommends `docker compose`.

---

## Initialization & Validation

### Check Version

```bash
docker-compose version
# Output:
# Docker Compose version 2.X.X
# Docker version 20.X.X
```

### Validate YAML Syntax

```bash
docker-compose config
# Outputs validated YAML if no errors
# Shows error messages if YAML invalid

# Example usage:
docker-compose config > validated.yml
# Outputs interpolated config to file
```

### List All Services

```bash
docker compose config --services
# Output:
# web
# db
# cache
```

### Create Compose File

**Windows:**
```bash
notepad docker-compose.yml
# Opens Notepad to create/edit file
```

**Linux/Mac:**
```bash
touch docker-compose.yml
nano docker-compose.yml
# Or use your preferred editor
```

---

## Starting Services

### Start All Services (Foreground)

```bash
docker-compose up
# Starts containers in foreground
# Displays logs from all services in real-time
# Press Ctrl+C to stop
```

**Output Example:**
```
web_1  | Starting Nginx server...
db_1   | PostgreSQL started on port 5432
cache_1 | Redis ready
web_1  | Listening on port 80
^C  # Stop by pressing Ctrl+C
```

### Start in Background (Detached)

```bash
docker-compose up -d
# Starts containers in background
# Returns immediately to prompt
# Check logs with: docker compose logs
```

### Rebuild Images Before Starting

```bash
docker-compose up --build
# Forces rebuild of all images
# Useful when Dockerfile changed
# Then starts containers
```

### Build Images Without Starting

```bash
docker-compose build
# Only builds images (doesn't start containers)
# Useful for CI/CD pipelines
```

### Force Rebuild (Ignore Cache)

```bash
docker-compose build --no-cache
# Builds from scratch without using cached layers
# Use when dependencies updated
```

### Scale Services

```bash
docker compose up --scale web=3
# Creates 3 instances of web service
# Named: web_1, web_2, web_3
# Useful for load testing or HA
```

**Example:**
```bash
# 3 web servers behind load balancer
docker compose up --scale web=3 -d

# Verify
docker compose ps
# web_1, web_2, web_3 all running
```

### Pull Latest Images

```bash
docker-compose pull
# Updates local images from registry
# Doesn't start containers
# Useful before deploying production

# Pull specific service
docker-compose pull web
# Only updates web service image
```

---

## Stopping Services

### Stop and Remove Everything (Full Cleanup)

```bash
docker-compose down
# Stops all containers
# Removes containers
# Removes networks created by compose
# KEEPS volumes (data preserved)

# Example:
docker-compose down
# Removing services and networks
# ✓ Cleanup complete
```

### Stop and Remove Volumes Too

```bash
docker-compose down -v
# Stops containers
# Removes containers and networks
# ALSO removes volumes (data DELETED)

# ⚠️ Use carefully - destroys data!
# Useful for fresh start or testing
```

### Stop Containers (Keep Them)

```bash
docker-compose stop
# Stops all running containers
# Containers still exist (not deleted)
# Can restart later with docker-compose start
```

**When to Use:**
- Temporary pause (keep state)
- Debugging (inspect container state)
- Maintenance (will restart later)

### Start Stopped Containers

```bash
docker-compose start
# Restarts previously stopped containers
# Data preserved (volumes intact)

# Start specific service
docker-compose start web
# Only restarts web service
```

### Restart All Containers

```bash
docker-compose restart
# Restarts all running containers
# Useful after config changes or fixing issues
# Data preserved

# Restart specific service
docker-compose restart db
# Only restarts database service
```

### Remove Stopped Containers

```bash
docker-compose rm
# Removes stopped containers
# Won't remove running containers
# Doesn't remove volumes

# Force remove running containers
docker-compose rm -f
# Removes even if still running
```

---

## Monitoring & Debugging

### List Running Services

```bash
docker-compose ps
# Shows status, ports, names of all services

# Output:
# NAME           COMMAND              STATUS      PORTS
# app_web_1      nginx -g daemon      Up 2 mins   0.0.0.0:3000→3000/tcp
# app_db_1       postgres             Up 2 mins   0.0.0.0:5432→5432/tcp
# app_cache_1    redis-server         Up 1 min    6379/tcp
```

**Column Explanation:**
| Column | Meaning |
|---|---|
| `NAME` | Container name |
| `COMMAND` | Container startup command |
| `STATUS` | Running/exited status, uptime |
| `PORTS` | Port mappings |

### View Logs (Buffered)

```bash
docker-compose logs
# Shows logs from all services
# Displays historical logs (buffered)
# Useful for post-debugging

# Last 100 lines
docker-compose logs --tail 100

# Specific service
docker-compose logs web
# Only logs from web service
```

### Stream Logs in Real-Time

```bash
docker-compose logs -f
# Streams logs from all services
# Updates as services log
# Like: tail -f logfile
# Press Ctrl+C to stop

# Stream specific service
docker-compose logs -f db
# Monitors only database service

# Stream with timestamps
docker-compose logs -f -t
# Includes timestamp with each log line
```

### Follow Multiple Services

```bash
docker-compose logs -f web db
# Stream logs from web and db only
# Useful to isolate relevant services
```

### View Last N Minutes

```bash
docker-compose logs --since 10m
# Shows logs from last 10 minutes

# Time options:
--since 1h      # Last 1 hour
--since 10m     # Last 10 minutes
--since 30s     # Last 30 seconds
```

---

## Executing Commands in Containers

### Enter Interactive Shell

```bash
docker-compose exec <service_name> bash
# Gives you shell access inside container
# Type commands interactively
# Exit with: exit or Ctrl+D

# Example:
docker-compose exec web bash
# Now inside web container:
root@container:/app# ls
root@container:/app# npm test
root@container:/app# exit
# Back on host
```

### Run One-off Commands

```bash
docker-compose exec <service> <command>

# Examples:
docker-compose exec app npm test
# Run tests

docker-compose exec db psql -U postgres -d mydb
# Access database

docker-compose exec web curl http://localhost:8000
# Check if service responds
```

### Without Allocating TTY (Non-interactive)

```bash
docker-compose exec -T app npm test
# For CI/CD pipelines
# Doesn't allocate interactive terminal
# Just runs command, exits
```

### Run as Different User

```bash
docker-compose exec -u root app apt-get update
# Run command as root (or other user)
# Useful for system commands
```

---

## Advanced Commands

### View Configuration

```bash
docker-compose config
# Shows validated, interpolated config
# All variables substituted
# Useful to verify variable substitution

# Example:
docker-compose config > /tmp/config.yml
# Save interpolated config to file
```

### List Images

```bash
docker-compose images
# Lists images used by services

# Output:
# REPOSITORY    TAG    IMAGE ID    SIZE
# nginx         1.20   a1234b56   150MB
# postgres      15     c7890d12   300MB
```

### Pause Running Containers

```bash
docker-compose pause
# Freezes containers (suspends execution)
# Useful for checkpoint/snapshot
# Resume with: docker-compose unpause

# Pause specific service
docker-compose pause web
```

### Resume Paused Containers

```bash
docker-compose unpause
# Resumes paused containers
# Continue from where they left off
```

### Kill Containers (Force Stop)

```bash
docker-compose kill
# Forcefully terminates containers
# No graceful shutdown
# Use when: normal stop not working, emergency

# Kill specific service
docker-compose kill web
```

### Push Images to Registry

```bash
docker-compose push
# Pushes built images to Docker registry
# Useful for CI/CD (save time rebuilding)
# Images must be tagged with registry

# Example:
# docker-compose.yml has:
# image: myregistry/myapp:latest
docker-compose push
# Pushes to myregistry
```

---

## Command Summary Table

| Command | Purpose | Example |
|---|---|---|
| `docker compose up -d` | Start all services in background | `docker compose up -d` |
| `docker compose down` | Stop + remove containers/networks | `docker compose down` |
| `docker compose down -v` | Stop + remove + delete volumes | `docker compose down -v` |
| `docker compose ps` | List running services and status | `docker compose ps` |
| `docker compose logs` | View all logs (buffered) | `docker compose logs` |
| `docker compose logs -f` | Stream logs in real-time | `docker compose logs -f` |
| `docker compose logs -f db` | Stream logs for specific service | `docker compose logs -f db` |
| `docker compose exec <svc> bash` | Enter container shell | `docker compose exec web bash` |
| `docker compose exec <svc> <cmd>` | Execute command in container | `docker compose exec app npm test` |
| `docker compose up --build` | Rebuild images and start | `docker compose up --build` |
| `docker compose build` | Build/rebuild images only | `docker compose build` |
| `docker compose up --scale web=3` | Run 3 instances of web | `docker compose up --scale web=3` |
| `docker compose stop` | Stop containers (keep them) | `docker compose stop` |
| `docker compose start` | Restart stopped containers | `docker compose start` |
| `docker compose restart` | Restart all running containers | `docker compose restart` |
| `docker compose config` | Validate and show YAML config | `docker compose config` |
| `docker compose pull` | Pull latest images from registry | `docker compose pull` |
| `docker compose kill` | Force stop containers | `docker compose kill` |
| `docker compose pause` | Pause running containers | `docker compose pause` |
| `docker compose unpause` | Resume paused containers | `docker compose unpause` |
| `docker compose rm` | Remove stopped containers | `docker compose rm` |
| `docker compose images` | List images used | `docker compose images` |

---

## Practical Workflows

### Development Workflow

```bash
# Day 1: Initial setup
git clone https://github.com/myproject
cd myproject
docker-compose up -d

# View logs
docker-compose logs -f

# While developing:
docker-compose restart web
# Restart after code changes

# Debug something
docker-compose exec db psql -U admin -d mydb

# Done for the day
docker-compose stop
```

### Production Deployment

```bash
# Pull latest code
git pull origin main

# Build new images
docker-compose build

# Start services with zero downtime (rolling update)
docker-compose up -d

# Verify services running
docker-compose ps

# Check logs for errors
docker-compose logs --since 1m

# If issue found, rollback:
git revert HEAD
docker-compose build
docker-compose up -d
```

### Testing Workflow

```bash
# Fresh environment
docker-compose down -v

# Start clean services
docker-compose up -d

# Run tests
docker-compose exec app npm test

# View detailed logs if tests fail
docker-compose logs app

# Cleanup
docker-compose down -v
```

### Debugging Workflow

```bash
# Check what's running
docker-compose ps

# Get detailed status
docker-compose logs -f

# Enter failing container
docker-compose exec <service> bash

# Inside container, debug:
# - Check environment: env
# - Check network: ping other_service
# - Check files: ls -la
# - Check processes: ps aux

# Exit and restart if modified
docker-compose restart <service>
```

---

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker-compose logs <service>

# Common issues:
# - Port already in use
# - Missing dependencies
# - Configuration error
# - Insufficient memory
```

### Can't Connect Between Services

```bash
# Verify services running
docker-compose ps

# Check network
docker-compose exec web ping db
# Should resolve db hostname

# Check port
docker-compose exec web telnet db 5432
```

### Data Lost After `docker-compose down`

```bash
# Check volumes
docker volume ls

# If down -v used:
docker-compose down -v
# This DELETES volumes and data!

# Always use: docker-compose down
# (without -v) for production
```

---

## Best Practices

1. **Always use `-d` flag in production**
   ```bash
   docker-compose up -d
   # Run in background, not foreground
   ```

2. **Use `depends_on` and health checks**
   ```yaml
   depends_on:
     db:
       condition: service_healthy
   # Ensures db ready before app starts
   ```

3. **Don't use in `docker-compose down -v` production**
   ```bash
   # This deletes volumes and data!
   # Use only for testing/development
   ```

4. **Stream logs, not history**
   ```bash
   docker-compose logs -f
   # Real-time monitoring better than historical logs
   ```

5. **Test locally before production**
   ```bash
   # Verify compose file works:
   docker-compose config
   docker-compose up -d
   docker-compose ps
   ```

---

## Next Topic

**[Read about Practical Examples →](08_Practical_Examples.md)**

Learn to build real-world applications with Docker Compose.
