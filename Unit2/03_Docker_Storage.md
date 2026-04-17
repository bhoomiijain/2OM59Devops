# Docker Storage, Volumes & Bind Mounts

The Problem with Containers:

By default: containers are ephemeral (temporary). All data inside lost when container removed.

Example:
docker run -it ubuntu bash
Inside: echo "Important Data" > /data/file.txt
exit → container stops
docker rm <container_id> → container removed

Result: file.txt gone forever. No recovery possible.

Real world problem:
Database container writes data to /var/lib/mysql.
Container deleted or crashed.
All data lost. Business disaster.

Solution: Store data outside container (persistent storage).

Two approaches:
1. Docker Volumes (managed by Docker, recommended)
2. Bind Mounts (manual management, good for development)

Volumes vs Bind Mounts:

Volumes:
- Managed by Docker
- Storage location: /var/lib/docker/volumes (Linux)
- Works across platforms: Yes (Windows, Mac, Linux)
- Backup: Easy
- Performance: Good
- Best for: Production
- Portability: Excellent

Bind Mounts:
- Managed by: You
- Storage location: Anywhere on host
- Works across platforms: Not portable
- Backup: Manual
- Performance: Lower
- Best for: Development
- Portability: Not recommended

Volumes (Recommended):

What is a volume?

Volume = storage location outside container, managed by Docker.

Container sees: /app/data (inside container)
Actually stored: /var/lib/docker/volumes/myvolume/_data (on host)
Mapping: /app/data → /var/lib/docker/volumes/myvolume/_data

When container removed: volume persists (data safe).

## Volume Management Commands

1. **docker volume create myvolume** : Create volume.

2. **docker volume ls** : List volumes.

3. **docker volume inspect myvolume** : Inspect volume details.
   - Shows: driver, mount point on host, creation time

4. **docker volume rm myvolume** : Remove volume.

5. **docker volume prune** : Remove all unused volumes.

6. **docker run -d -v myvolume:/app/data nginx** : Run container with volume.
   - This maps myvolume → /app/data inside container

Volume Examples:

## Volume Examples

**Example 1: Database persistence**

Create volume:
```bash
docker volume create mysql_data
```

Run MySQL with volume:
```bash
docker run -d \
  --name mysql_db \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -v mysql_data:/var/lib/mysql \
  mysql:8
```

Reuse volume in new container:
```bash
# Database data stored in mysql_data volume
# Even if container deleted, data persists in volume
# Create new container, reuse volume:
docker run -d \
  --name mysql_db_new \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -v mysql_data:/var/lib/mysql \
  mysql:8
# New container has all old data!
```

## Bind Mounts (Manual Host Management)

**What is bind mount?**

- Bind mounts = manual management (you manage path, not Docker)
- Stored anywhere on host filesystem
- Not portable across platforms
- Good for development

**Basic syntax**:
```bash
docker run -d -v /host/path:/container/path image:tag
```

**Examples**:

```bash
# Linux/Mac - current directory
docker run -d -v $(pwd):/app ubuntu

# Windows - host path to container
docker run -d -v C:\app\code:/app ubuntu

# Mount single file
docker run -d -v ~/.bashrc:/root/.bashrc ubuntu

# Read-only mount
docker run -d -v /host/path:/container/path:ro ubuntu
```

docker run -d \
  --name webserver \
  -p 8080:80 \
  -v website_data:/usr/share/nginx/html \
  nginx

Create file inside container:
docker exec webserver bash -c "echo '<h1>My Site</h1>' > /usr/share/nginx/html/index.html"

Stop and remove container:
docker stop webserver
docker rm webserver

Start new container with same volume:
docker run -d \
  --name webserver_2 \
  -p 8080:80 \
  -v website_data:/usr/share/nginx/html \
  nginx

File still there! Data persisted.

Example 3: Multiple containers sharing volume

docker volume create shared_data

Container 1 writes data:
docker run -it --name writer -v shared_data:/shared ubuntu bash
Inside: echo "Hello from writer" > /shared/message.txt

Container 2 reads data:
docker run -it --name reader -v shared_data:/shared ubuntu bash
Inside: cat /shared/message.txt
Output: Hello from writer

Example 4: Named volumes in docker-compose

volumes:
  db_data:
    driver: local

services:
  database:
    image: mysql:8
    volumes:
      - db_data:/var/lib/mysql

Bind Mounts:

What is bind mount?

Bind mount = mount host directory into container.
Host path stays under your control.
Great for development (edit code on host, changes reflect in container immediately).

Syntax:
docker run -d -v /host/path:/container/path image:tag

Windows:
docker run -d -v C:\Users\Dell\app:/app nginx

Linux/Mac:
docker run -d -v $(pwd):/app nginx

Bind Mount Examples:

Example 1: Development workflow

Create project directory:
mkdir my_app
cd my_app

Create app.js:
echo "console.log('Hello Docker');" > app.js

Run container with bind mount:
docker run -d \
  -v $(pwd):/app \
  -w /app \
  --name dev_app \
  node:18 \
  node app.js

Edit app.js on host:
echo "console.log('Updated');" > app.js

Change reflected in container immediately (no rebuild needed).

Example 2: Web development

host directory: ~/website/html (contains index.html)
container: /usr/share/nginx/html

docker run -d \
  -p 8080:80 \
  -v ~/website/html:/usr/share/nginx/html \
  --name dev_web \
  nginx

Edit ~/website/html/index.html on host.
Browser refresh → changes visible instantly.
No rebuild, no redeployment.

Example 3: Configuration mounting

Config file on host: /etc/app/config.json
Container: /app/config.json

docker run -d \
  -v /etc/app/config.json:/app/config.json:ro \
  nginx

:ro = read-only. Container reads config but can't modify.

Example 4: Logs collection

Mount host logs directory:
docker run -d \
  -v /var/log/myapp:/app/logs \
  myapp

Application writes logs to /app/logs (inside container).
Logs accessible from host at /var/log/myapp.
Easy to collect/analyze logs from host.

Volume vs Bind Mount Comparison:

Use Volumes When:
- Production environments
- Database persistence (MySQL, MongoDB, PostgreSQL)
- Want Docker to manage storage
- Multi-container data sharing
- Cross-platform portability needed
- Easier backup/restore

Use Bind Mounts When:
- Development (code editing)
- Configuration file passing
- Want direct host access
- Performance not critical
- Don't need cross-platform support

Volume Drivers:

Local driver (default):
docker volume create -d local myvolume

Other drivers available:
- nfs: network file system
- local-persist: persistent local
- aws: AWS EBS volumes

Example with NFS:
docker volume create -d nfs -o addr=192.168.1.100,vers=4 nfs_vol

Container Data Lifecycle:

Scenario: MySQL database

1. Create volume:
docker volume create mysql_data

2. Run MySQL container:
docker run -d \
  --name mysql_prod \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -v mysql_data:/var/lib/mysql \
  mysql:8

3. Create database and tables:
docker exec mysql_prod mysql -u root -proot123 -e "CREATE DATABASE myapp;"

4. Container running, data safe in mysql_data

5. Stop container (data persists):
docker stop mysql_prod

6. Remove container (data still persists):
docker rm mysql_prod

7. Start NEW container with same volume:
docker run -d \
  --name mysql_recovery \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -v mysql_data:/var/lib/mysql \
  mysql:8

8. Database and tables still there! Data recovered!

Complete Multi-Container Example with Volumes:

docker volume create db_data
docker volume create app_data

docker network create app_net

docker run -d \
  --name database \
  --network app_net \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=myapp \
  -v db_data:/var/lib/mysql \
  mysql:8

docker run -d \
  --name app \
  --network app_net \
  -e DB_HOST=database \
  -v app_data:/app/config \
  myapp:v1

docker run -d \
  --name web \
  --network app_net \
  -p 8080:80 \
  -v app_data:/usr/share/nginx/html \
  nginx

All three containers share networks and volumes for complete system.

Best Practices:

1. Use volumes for persistent data (production)
2. Use bind mounts for development
3. Always backup important volumes
4. Document which volumes containers use
5. Clean up unused volumes regularly (docker volume prune)
6. Use descriptive volume names
7. Separate volumes for different data types
8. Consider backup strategy before deployment

Data persists:
Container A writes to volume → Container A deleted → volume still exists
Container B created → same volume mounted → Container B reads Container A's data

Use case: databases, persistent caches, user uploads

Advantages of volumes:

1. Persistence
Container deleted = data survives. Reattach volume to new container.

2. Sharing between containers
Multiple containers mount same volume = shared storage.

Example:
Web server stores uploaded files in volume.
Another worker container processes files from same volume.

3. Easier backup/restore
docker volume inspect myvolume (find location)
tar -czf backup.tar.gz /var/lib/docker/volumes/myvolume/_data (backup)
Extract to restore

4. Performance
Optimized for container environments, faster than bind mounts on some systems.

5. Driver options
Use different volume drivers (local, NFS, AWS EBS, etc).

Why containers need volumes:

Container layers are Copy-on-Write (ephemeral top layer). When container stops:
- Changes to lower layers: permanent (in image)
- Changes to top writeable layer: deleted (container-level data)

Volume changes: permanent (stored outside container filesystem)

Database scenario:

MySQL running in container.
App writes to /var/lib/mysql inside container.
Without volume: container deleted → database gone
With volume: container deleted → /var/lib/mysql data persists in volume

Docker Volume Commands:

Create volume:
docker volume create myvolume
Creates volume named 'myvolume' managed by Docker.

List volumes:
docker volume ls
Shows all volumes with driver (local usually).

Inspect volume:
docker volume inspect myvolume
JSON output showing:
- Driver
- Mountpoint (where on host)
- Labels
- Scope
- Creation date

Remove volume:
docker volume rm myvolume
Deletes volume (and all data inside!). Use carefully.

Remove unused volumes:
docker volume prune
Cleans up volumes not attached to any container.

Using Volumes with Containers:

Syntax:
docker run -v <volume_name>:<path_inside_container> image

Example: Mount volume in Ubuntu
docker run -it -v myvolume:/app/data ubuntu bash

Inside container:
cd /app/data
echo "Hello Volume" > file.txt
cat file.txt
exit

Stop and remove container:
docker stop <container_id>
docker rm <container_id>

Data persists in volume.

Create new container with same volume:
docker run -it -v myvolume:/app/data ubuntu bash

Inside container:
cat /app/data/file.txt
Output: Hello Volume (data recovered!)

Multiple containers sharing volume:

Container A (Writer):
docker run -d --name writer -v shared_data:/data ubuntu bash -c "echo 'Data from A' > /data/dataA.txt && sleep 1000"

Container B (Reader):
docker run -it --name reader -v shared_data:/data ubuntu bash

Inside B:
cat /data/dataA.txt
Output: Data from A

Both see same volume data.

Production Example - MySQL with Volume:

Create volume:
docker volume create mysql_data

Run MySQL with volume:
docker run -d \
  --name mysql_prod \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=production \
  -v mysql_data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8

Data written by MySQL to /var/lib/mysql → actually stored in volume → persists.

Even if you:
docker stop mysql_prod
docker rm mysql_prod

Data safe in mysql_data volume.

Recreate container with same volume:
docker run -d \
  --name mysql_prod \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=production \
  -v mysql_data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8

All data back. Database intact.

Backup MySQL volume:
docker run --rm -v mysql_data:/dbdata -v $(pwd):/backup ubuntu tar czf /backup/mysql_backup.tar.gz -C /dbdata .

Creates compressed backup.

Bind Mounts:

What is a bind mount?

Bind mount = link specific folder on host directly into container.

Container sees: /app (inside container)
Actually is: /home/user/myapp (on host)
Mapping: /app (container) → /home/user/myapp (host)

Files on host instantly visible inside container, and vice versa.

Use cases:

1. Development
hot reload code without rebuilding image

2. Configuration
manage config files on host, container reads them

3. Logging
container writes logs to host mount point

4. Direct data access
container can access files on host directly

Bind Mount Syntax:

docker run -v <host_path>:<container_path> image

Absolute path required (can't use relative).

Linux/Mac:
docker run -v /home/user/app:/app ubuntu

Windows:
docker run -v C:\Users\user\app:/app ubuntu

Read-only bind mount:
docker run -v /host/path:/container/path:ro image

Container can read but not modify.

Bind Mount for Development:

Node.js development with live reload:

Create project:
mkdir myapp
cd myapp
echo 'console.log("Hello");' > app.js

Dockerfile:
FROM node:18
WORKDIR /app
CMD ["node", "app.js"]

Build:
docker build -t myapp-dev .

Run with bind mount (current directory):
docker run -it -v $(pwd):/app myapp-dev

Inside or on host, edit app.js:
echo 'console.log("Hello Updated");' > app.js

Run again (without rebuild):
docker run -it -v $(pwd):/app myapp-dev

Sees updated code immediately.

Nginx with bind mount (custom content):

Create html folder:
mkdir html
echo '<h1>Hello from Bind Mount</h1>' > html/index.html

Run:
docker run -d -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html nginx

Edit html/index.html on host:
echo '<h1>Updated Content</h1>' > html/index.html

Refresh browser: http://localhost:8080
Sees updated content instantly (no container restart).

Copy-on-Write (CoW) in Docker Filesystems:

Docker images use layered filesystem (OverlayFS on Linux).

Image layer structure:
Bottom (Base layer): Ubuntu OS (read-only)
Middle (App layer): Application (read-only)
Top (Container layer): Writable layer (unique per container)

How Copy-on-Write works:

1. Read operation:
Container reads /etc/config file.
File in base layer (read-only).
Docker returns it directly (fast, shared).

2. Write/modify operation:
Container edits /etc/config.
Docker:
- Copies file from read-only layer to writable layer
- Modifies copy
- Original stays unchanged
- Other containers still see original

Example:
Two containers from same image.
Container A modifies file → copied to Container A's layer
Container B still sees original → no copied file needed
Storage = shared base + small unique copies per container

Benefit: efficiency. Without CoW:
Image = 600MB
5 containers = 5 × 600MB = 3000MB storage

With CoW:
Image = 600MB (shared)
5 containers = 5 × 10MB (only changed parts) = 50MB
Total = 650MB (instead of 3050MB)

Viewing layers:

docker history nginx
Shows each layer (instruction, size, created).

docker inspect nginx
Shows layer details (JSON format).

Volumes Bypass CoW:

Volume data doesn't use CoW. Data written to volume = persistent directly.

docker run -v myvolume:/data ubuntu
Writing to /data → directly to volume storage
Not affected by CoW (ephemeral layer)
Survives container deletion

Combining Volumes and Bind Mounts:

Single container with both:
docker run -it \
  -v mysql_data:/var/lib/mysql \
  -v $(pwd)/config:/etc/mysql/conf.d \
  mysql:8

/var/lib/mysql = volume (persistent database)
/etc/mysql/conf.d = bind mount (host config files)

Real-World Example: Full Stack App with Shared Volume

Create shared volume:
docker volume create app_shared

Database container:
docker run -d \
  --name db \
  -v app_shared:/app \
  -e MYSQL_ROOT_PASSWORD=root \
  mysql:8

Application container:
docker run -d \
  --name app \
  -v app_shared:/app \
  -p 3000:3000 \
  node-app:v1

Log container (reads from shared volume):
docker run -d \
  --name logger \
  -v app_shared:/app \
  busybox tail -f /app/logs

All three containers share same volume. High coupling but real-time data sharing.

Best Practices for Storage:

1. Use volumes for production databases
2. Use volumes for stateful services
3. Use bind mounts for development only
4. Keep important data on volumes or backups
5. Regular volume backups
6. Use read-only bind mounts when possible (prevent accidental modifications)
7. Document which containers use which volumes

Volume lifecycle management:

docker volume ls (see all)
docker volume inspect vol1 (details)
docker volume rm vol1 (delete - careful!)
docker volume prune (cleanup unused)

Use named volumes in production (easy tracking).
Avoid anonymous volumes (hard to manage).

Storage Commands Summary:

Volume commands:
docker volume create name
docker volume ls
docker volume inspect name
docker volume rm name
docker volume prune

Mount volume during run:
docker run -v vol_name:/path image

Mount bind during run:
docker run -v /host/path:/container/path image

Read-only mount:
docker run -v /path:/path:ro image
  --name mysql_db_new \
  -e MYSQL_ROOT_PASSWORD=root \
  -v mysql_data:/var/lib/mysql \
  mysql:8
```

## Hands-on Lab 1 — Volume Persistence (from class)

```bash
# Step 1: Create volume
docker volume create projectdata
docker volume ls
docker volume inspect projectdata

# Step 2: Run container with volume attached
docker run -dit --name project_container -v projectdata:/app/data ubuntu

# Step 3: Write a file inside the container
docker exec -it project_container sh -c "echo 'This is my first volume' > /app/data/report.txt"
docker exec -it project_container cat /app/data/report.txt

# Step 4: Stop and DELETE the container
docker stop project_container
docker rm project_container

# Step 5: Run a NEW container with the SAME volume
docker run -dit --name project_container_new -v projectdata:/app/data ubuntu

# Step 6: Verify data is still there
docker exec -it project_container_new ls /app/data
docker exec -it project_container_new cat /app/data/report.txt
# → This is my first volume  ✅ Data persisted!
```

---

## Hands-on Lab 2 — Full University Portal (from class)

```bash
# Deploy Apache portal with volume + env variable
docker pull httpd
docker volume create portaldata

docker run -dit \
  --name college_portal \
  -p 8080:80 \
  -e ENV=production \
  -v portaldata:/usr/local/apache2/htdocs \
  httpd

# Modify the web page
docker exec -it college_portal bash -c \
  "echo '<h1>Welcome to University Portal - Production Mode</h1>' \
  > /usr/local/apache2/htdocs/index.html"

# Verify
curl http://localhost:8080

# Check logs
docker logs college_portal
docker logs -f college_portal   # follow live

# Cleanup
docker stop college_portal
docker rm college_portal
docker volume rm portaldata
docker rmi httpd
```

---

## Hands-on Lab 3 — Bind Mount with Ubuntu (from class)

```bash
# Create directory on host
mkdir -p ~/app/data       # Linux
# OR: mkdir C:\app\data   # Windows

# Run container with bind mount + env variable
docker run -it \
  --name my_app \
  -e APP_ENV=production \
  -v ~/app/data:/data \
  ubuntu bash

# Inside container:
echo $APP_ENV                          # → production
echo "Hello" > /data/test.txt
ls /data
cat /data/test.txt
exit

# On host — file is visible:
ls ~/app/data
# → test.txt  ✅

# Container deleted — file stays on host
docker rm -f my_app
ls ~/app/data    # → test.txt still there ✅
```

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: New PPT slides 41–53, 63–68 (Basic_Docker_commands + Unit_2)*

 
