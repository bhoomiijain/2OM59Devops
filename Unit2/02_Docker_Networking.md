# Docker Networking

Why Docker Networking?

By default: containers are isolated. Can't talk to each other or outside world.

Docker networking enables:
- Container-to-container communication
- Container reached from browser
- Applications coordinate across containers
- Service discovery by name
- Controlled network segregation

Problem without networking:
Container A (web server) needs to reach Container B (database).
Without networking: Container A can't reach B. Web server crashes (can't connect to DB).
With networking: Container A reaches B by name or IP. Works!

Network Basics:

Docker creates isolated networks. Containers attached to network communicate.

Default networks (created automatically):
- bridge: default for docker run
- host: uses host machine network directly
- none: no network access

Custom networks: user-defined, best practice for production.

A. Bridge Network (Default)

Default network when container runs without specifying network.

How bridge works:
Host machine has physical network (192.168.1.0/24).
Docker daemon creates virtual bridge interface (docker0).
Each container gets IP on bridge network (172.17.0.2, 172.17.0.3, etc).
Containers on same bridge can communicate by IP.
Containers on default bridge cannot resolve names (no DNS).

Default bridge network commands:

docker network ls
List all networks. Shows bridge, host, none networks.

docker network inspect bridge
Show detailed info about default bridge:
- Subnet (172.17.0.0/16)
- Connected containers
- Gateways
- Driver info

Containers on default bridge:
docker run -d --name container_a nginx
docker run -d --name container_b ubuntu sleep 1000

Both on default bridge (docker0).

Test connectivity by IP:
docker exec container_b ping 172.17.0.2 (IP of container A)
For default bridge, containers don't resolve names (container_a won't work).

Issues with default bridge:
- No DNS name resolution between containers
- Limited isolation control
- Not recommended for production

B. Custom Bridge Network (RECOMMENDED FOR PRODUCTION)

Better than default bridge. Provides:
- Automatic DNS resolution (ping by container name)
- Better isolation
- Easier management

Create custom bridge:
docker network create my_bridge_net

Run containers on custom network:
docker run -d --name container_a --network my_bridge_net nginx
docker run -d --name container_b --network my_bridge_net ubuntu sleep 1000

Test connectivity by name (DNS works):
docker exec container_b ping container_a
Resolves to IP automatically!

Multiple networks:
docker network create net1
docker network create net2

docker run -d --name app1 --network net1 nginx
docker run -d --name app2 --network net2 nginx

app1 and app2 cannot communicate (different networks).

Connect container to additional network:
docker network connect net2 app1
app1 now on both net1 and net2 (can communicate with both).

Inspect custom network:
docker network inspect my_bridge_net
Shows connected containers, subnet, IP assignments.

Remove network:
docker network rm my_bridge_net
(Only works if no containers attached)

C. Host Network

Container shares host's networking stack (not isolated).
No separate IP - uses host IP directly.
No need for port mapping (-p not needed).
Faster performance (no virtual bridge).
Linux only (not available on Docker Desktop for Windows/Mac).

When to use:
- Performance critical
- Need direct host access
- Monitoring/system tools

Example:
docker run -d --network host nginx

Access nginx directly on host IP:
curl http://localhost:80 (works without -p)

No port mapping needed with host network.

D. Overlay Network

Used in Docker Swarm mode for multi-host deployments.
Connects containers across multiple Docker hosts.
Requires Swarm mode enabled.

Initialize swarm:
docker swarm init

Create overlay network:
docker network create -d overlay my_overlay

Deploy service on overlay:
docker service create --name web --network my_overlay -p 8080:80 nginx

Containers on different hosts can communicate across overlay network.

Advanced: DNS Inside Docker

Docker has built-in DNS (127.0.0.11:53).

Automatic DNS resolution:
Containers on custom network resolve each other by name.
Containers resolve Docker host by name "host.docker.internal" (Windows/Mac).

Example:
docker network create app_net

docker run -d --name backend --network app_net nginx
docker run -it --name client --network app_net alpine sh

Inside client container:
ping backend → resolves to IP automatically!
ping host.docker.internal → reaches host machine

Service Discovery:
Instead of hardcoding IPs in config files:
Before (hard coded IP):
app connects to 172.17.0.2:3306 (database IP)

After (service discovery):
app connects to database:3306 (resolves by name)
Easy: stop/restart database in different IP, app still works.

Container Linking (Legacy - Deprecated)

Old method: --link flag.
Before custom networks, Docker used --link.
Obsolete - use custom networks instead.

Example (old way - DON'T USE):
docker run -d --name db nginx
docker run -it --name app --link db:database alpine sh
Inside app: ping database

Modern way (USE THIS):
docker network create app_net
docker run -d --name db --network app_net nginx
docker run -it --name app --network app_net alpine sh
Inside app: ping db

Custom network is better, more flexible.

Complete Networking Example:

1. Create custom network:
docker network create app_bridge

2. Run multiple containers on same network:
docker run -d --name mysql_db --network app_bridge -e MYSQL_ROOT_PASSWORD=root mysql:8
docker run -d --name backend_app --network app_bridge -e DB_HOST=mysql_db node:18

3. backend_app automatically connects to mysql_db by name (DNS resolution).

4. Verify communication:
docker exec backend_app ping mysql_db
Docker resolves mysql_db to IP automatically

5. Expose web server with port mapping:
docker run -d --name web --network app_bridge -p 8080:80 nginx

Port mapping makes nginx accessible from host.

6. All containers communicate internally by name in app_bridge network.

Practical Multi-Container Application:

docker network create webapp_net

docker run -d \
  --name database \
  --network webapp_net \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=webapp \
  mysql:8

docker run -d \
  --name cache \
  --network webapp_net \
  redis:7

docker run -d \
  --name api \
  --network webapp_net \
  -e DB_HOST=database \
  -e CACHE_HOST=cache \
  myapi:v1

docker run -d \
  --name frontend \
  --network webapp_net \
  -p 8080:80 \
  -e API_HOST=api \
  nginx

All containers communicate by name within webapp_net network:
- frontend connects to api:80
- api connects to database:3306 and cache:6379
- No hardcoded IPs needed

Network Best Practices:

1. Use custom bridge networks (not default bridge)
2. Name containers for DNS resolution
3. Use environment variables for hostnames
4. One network per application
5. Isolate different applications in different networks
6. Use unique container names within network
7. Document network topology (which containers on which networks)

Limitation of default bridge:
- DNS by container name doesn't work. Must use IP.
- container_b: ping container_a (fails)
- container_b: ping 172.17.0.2 (works)

Limitation discovered: must use IP, can't use name. Not ideal for microservices.

Custom Bridge Networks:

Create custom bridge:
docker network create app_bridge

Run containers on custom bridge:
docker run -d --name web1 --network app_bridge nginx
docker run -d --name db1 --network app_bridge ubuntu sleep 1000

Test connectivity by name:
docker exec -it db1 ping web1
Result: works! (DNS resolves "web1" to container IP)

Why custom bridge better:
- Automatic DNS by container name
- Better isolation (multiple bridge networks separate)
- Easy container communication
- Easier to manage

Custom Bridge Commands:

Create network:
docker network create my_app_net

List networks:
docker network ls
Shows: bridge (default), host, none, and user-defined networks.

Inspect network:
docker network inspect my_app_net
Shows: containers on network, IP range, driver, etc.

Delete network:
docker network rm my_app_net

Connect container to network:
docker network connect my_app_net running_container

Disconnect container from network:
docker network disconnect my_app_net running_container

Run container on specific network:
docker run -d --name myapp --network my_app_net nginx

Container Communication on Custom Bridge:

Architecture:
Container A (IP: 172.20.0.2) --- Bridge Network --- Container B (IP: 172.20.0.3)

Container A can reach B by:
- ping 172.20.0.3 (by IP)
- ping containerb (by container name - Docker DNS)

Example - Web Tier and DB Tier:

Create app network:
docker network create app_tier

Run database container:
docker run -d \
  --name postgres_db \
  --network app_tier \
  -e POSTGRES_PASSWORD=dbpass \
  postgres:15

Run web container (references DB by name):
docker run -d \
  --name webapp \
  --network app_tier \
  -e DATABASE_URL=postgresql://postgres_db:5432/mydb \
  -p 8080:5000 \
  mywebapp:v1

Inside webapp: can reach DB using "postgres_db" hostname (Docker DNS).

B. Host Network

Container uses host machine network stack directly.

No container IP. No port mapping needed.

When to use:
- Performance critical (no bridging overhead)
- Need access to host services directly
- Linux only (not Docker Desktop)

Example:
docker run -d --network host nginx

Nginx listens on host's port 80 directly.
No port mapping needed.
browser → localhost:80 → Nginx (works directly)

Contrast with bridge:
Bridge: docker run -d -p 8080:80 nginx (must map)
Host: docker run -d --network host nginx (no mapping, direct)

Limitation: Linux only. Docker Desktop (Windows/Mac) doesn't support.

C. Overlay Network

Used in Docker Swarm (multi-host orchestration).

Connects containers on different physical/virtual machines.

Encrypted by default. Suitable for production clusters.

When to use:
- Multiple Docker hosts (machines)
- Services span across hosts
- Docker Swarm mode

Example (simplified):
docker swarm init (enable swarm mode)
docker network create -d overlay my_overlay
docker service create --network my_overlay --name web nginx

Containers on different hosts can communicate through overlay.

Not covered in detail here (requires Swarm setup).

D. None Network

Container has zero network access.

Maximum isolation.

When to use:
- Testing network isolation
- Offline processing
- Security sensitive processes

Example:
docker run --network none ubuntu sleep 1000

Container has no network access. Can't reach outside.

Port Mapping (-p Flag)

Without port mapping:
docker run -d nginx (runs inside, not accessible from browser)

Port mapping exposes container port to host:

Syntax: -p <HOST_PORT>:<CONTAINER_PORT>

Example:
docker run -d -p 8080:80 nginx

Flow:
Browser → localhost:8080 → Docker Host (port 8080) → Bridge → Container → port 80

Multiple ports:
docker run -d -p 8080:80 -p 443:443 nginx

Container listening on 80 and 443, exposed as host 8080 and 443.

Port mapping with databases:

MySQL (container port 3306):
docker run -d -p 3307:3306 --name mysql_dev -e MYSQL_ROOT_PASSWORD=root mysql:8

Access from host: localhost:3307

PostgreSQL (container port 5432):
docker run -d -p 5433:5432 --name postgres_dev postgres:15

Access from host: localhost:5433

MongoDB (container port 27017):
docker run -d -p 27017:27017 --name mongo_dev mongo

Access from host: localhost:27017

Port mapping scenarios:

Scenario 1 - Single container:
docker run -d -p 8080:80 nginx
Host 8080 → Container 80

Scenario 2 - Multiple containers, same container port:
docker run -d -p 8080:80 nginx (container 1, host 8080)
docker run -d -p 8081:80 nginx (container 2, host 8081)

Both run Nginx (port 80), but exposed differently.

Scenario 3 - Container port forwarding from specific interface:
docker run -d -p 127.0.0.1:8080:80 nginx

Only accessible from localhost (127.0.0.1), not from other machines.

docker run -d -p 0.0.0.0:8080:80 nginx

Accessible from any machine on network.

DNS and Service Discovery:

Container nameresolution:

On custom networks:
Docker daemon runs DNS server (127.0.0.11:53).

When container queries hostname:
docker exec -it container1 ping container2

DNS resolves "container2" to its IP on that network.

Service discovery by name:

Create network:
docker network create services

Run containers:
docker run -d --name api --network services api-service:v1
docker run -d --name db --network services postgres

API container can reach DB:
docker exec -it api curl http://db:5432 (uses DNS to resolve "db")

DNS enables loose coupling:
API doesn't care about DB IP.
IP can change, DNS still resolves by name.

Container aliases:

Run container with alias:
docker run -d --network mynet --name db --link api:db_link postgres

Other containers can reach it as "db_link"

Linking Containers (Legacy - Deprecated)

Old way using --link flag:

docker run -d --name db postgres
docker run -d --name web --link db:database nginx

"web" container can reach "db" container as "database"

Problem: only one direction, complex setup.

Modern way (recommended):

Use custom networks instead:
docker network create app_net
docker run -d --name db --network app_net postgres
docker run -d --name web --network app_net nginx

"web" and "db" can reach each other by name automatically.

Better: simpler, bidirectional, standard practice.

Docker Network Commands - Full Reference:

docker network ls
List all networks (bridge, host, none, custom).

docker network create <name>
Create new bridge network.

docker network create -d overlay <name>
Create overlay network (Swarm mode).

docker network inspect <network_name>
Show detailed network info (containers, IP range, driver).

docker network connect <network_name> <container>
Add running container to network.

docker network disconnect <network_name> <container>
Remove container from network.

docker network rm <network_name>
Delete network.

docker network prune
Remove unused networks.

Practical Multi-Container Application:

Setup three-tier application with containers.

Network:
docker network create three_tier

Database layer:
docker run -d \
  --name postgres_db \
  --network three_tier \
  -e POSTGRES_PASSWORD=dbpass \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:15

Backend API layer:
docker run -d \
  --name api_server \
  --network three_tier \
  -e DATABASE_URL=postgresql://postgres:dbpass@postgres_db:5432/appdb \
  -p 5000:5000 \
  backend-api:v1

Frontend layer:
docker run -d \
  --name web_frontend \
  --network three_tier \
  -e API_URL=http://api_server:5000 \
  -p 80:80 \
  frontend-app:v1

How they communicate:
- Browser → localhost:80 → frontend (container, port 80)
- frontend container → api_server hostname (Docker DNS) → backend (container, port 5000)
- backend → postgres_db hostname → database (container, port 5432)
- All on same three_tier network

Benefits:
- Service discovery by name
- Isolated network from default bridge
- Easy to manage (one command to remove all three)

Network Drivers:

bridge: default, single-host communication, good for development
host: container uses host network, Linux only, high performance
overlay: multi-host, Swarm mode, encrypted
macvlan: container gets MAC address on physical network, enterprise setups
ipvlan: container gets IP on physical network
null (none): no network

Default driver for custom networks: bridge

Specifying driver:
docker network create -d bridge my_bridge
docker network create -d overlay my_overlay (requires Swarm)

IP Address Management:

Custom bridge with specific IP range:
docker network create --subnet=192.168.0.0/16 my_net

Run container with specific IP:
docker run -d --network my_net --ip 192.168.0.10 nginx

Docker IPAM (IP Address Management):
- Allocates IPs from subnet
- Prevents IP conflicts
- Allows custom IP assignment

Network Troubleshooting:

Test connectivity between containers:
docker exec -it container_a ping container_b

Inspect which network container is on:
docker inspect <container_name> | grep -A 10 NetworkSettings

View all containers on network:
docker network inspect <network_name>

Check container IP:
docker exec -it <container_name> hostname -i

Best Practices for Docker Networking:

1. Use custom networks for production
2. Use DNS by container name (avoid hardcoding IPs)
3. Run related containers on same network
4. Isolate networks by tier/environment
5. Use port mapping for external access only
6. Don't use default bridge in production
7. Document network topology
8. Use overlay networks for multi-host (Swarm/Kubernetes)
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

