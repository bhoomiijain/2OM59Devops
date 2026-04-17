# Docker Networking

Why Docker Networking?
By default containers isolated - can't talk to each other or outside world without setup. Docker networking allows:
- Containers communicate with each other
- Containers reached from browser/host
- Controlled isolation between services
- DNS-based service discovery by container name

Default Networks (auto-created by Docker):
docker network ls shows:
- bridge: default network when you run container
- host: container network stack shared with host
- none: no network access

A. Bridge Network (Default)
- Default when you run any container
- Containers on same bridge can communicate via IP
- Best for standalone containers on single host
- Custom bridge networks support DNS by name

Commands:
docker network ls = list networks
docker network inspect bridge = inspect default bridge
docker network create my_bridge_net = create custom bridge
docker run -d --name container1 --network my_bridge_net nginx = run on custom bridge
docker run -d --name container2 --network my_bridge_net alpine sleep 300
docker exec -it container2 ping container1 = test connectivity

Important: DNS by container name only works on custom bridges, not default bridge.

B. Host Network
- Container shares host machine network stack directly
- No separate IP for container
- No need for -p port mapping - app listens on host ports directly
- Linux only (not Docker Desktop Windows/Mac)
- Faster but less isolation

docker run -d --network host nginx
curl http://localhost = works directly, no mapping needed

C. Overlay Network
- Used in Docker Swarm (multi-host setup)
- Connects containers on different physical/virtual hosts
- Encrypted by default

docker swarm init = initialize Swarm first
docker network create -d overlay my_overlay = create overlay

D. None Network
- Container has zero network access
- Maximum isolation

docker run --network none ubuntu

DNS Inside Docker
Docker has built-in DNS server for user-defined networks. Containers on same custom network reach each other by container name. Works only on user-defined bridge networks (not default).

docker network create app_net = create custom network
docker run -d --name backend --network app_net nginx = run backend on network
docker run -it --name client --network app_net alpine sh = run client on network
Inside client: ping backend = works because Docker DNS resolves name

Port Mapping (Host ↔ Container)
Without -p containers not accessible from browser or host.

Syntax: -p <HOST_PORT>:<CONTAINER_PORT>

docker run -d -p 8080:80 --name mynginx nginx = map host 8080 to container 80
docker run -d -p 8080:80 -p 443:443 nginx = multiple ports
docker run -d -p 3307:3306 --name mysql-test -e MYSQL_ROOT_PASSWORD=root123 mysql:8 = MySQL: host 3307 to container 3306

Internal flow:
Browser → localhost:8080 → Docker Host → Container:80 → Nginx

## Linking Containers (Legacy — Deprecated)

The old `--link` way. **Don't use this** — use custom networks instead.

```bash
# OLD WAY (deprecated)
docker run -d --name db nginx
docker run -it --name app --link db:database alpine sh

# MODERN WAY (use custom network)
docker network create appnet
docker run -d --name db --network appnet nginx
docker run -it --name app --network appnet alpine sh
# Inside app: ping db    ← works by DNS
```

---

## Network Commands — Full Reference

```bash
# List all networks
docker network ls

# Create a network
docker network create mynetwork

# Inspect network
docker network inspect mynetwork

# Delete network
docker network rm mynetwork

# Connect a running container to a network
docker network connect mynetwork mycontainer

# Disconnect
docker network disconnect mynetwork mycontainer
```

---

## Complete Multi-Container Example (from class)

```bash
# 1. Create shared network
docker network create app_bridge

# 2. Create volume for MySQL data
docker volume create mysql_data

# 3. Run MySQL with network + volume
docker run -d \
  --name mysql_db \
  --network app_bridge \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=myapp \
  -v mysql_data:/var/lib/mysql \
  mysql:8

# 4. Run backend connected to same network
docker run -d \
  --name backend_app \
  --network app_bridge \
  -e DB_HOST=mysql_db \
  node:18-alpine sh -c "apk add curl && sleep 300"

# 5. Test connectivity (backend can reach mysql_db by name)
docker exec -it backend_app ping mysql_db

# 6. Run frontend on same network + expose to host
docker run -d \
  --name frontend \
  --network app_bridge \
  -p 8080:80 \
  nginx
```

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: New PPT slides 57–63, 66–72 (Basic_Docker_commands + Unit_2)*

