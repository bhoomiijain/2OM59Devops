# Docker Storage

The Problem:
Containers emphemeral - all data inside lost when container removed.

docker rm mycontainer = all data gone

To persist data across container restarts/deletions:
1. Volumes - managed by Docker
2. Bind Mounts - managed by you

1. Docker Volumes
Volumes managed entirely by Docker. Stored outside container filesystem.

Default storage: /var/lib/docker/volumes/

Why use volumes?
- Data persists after container deleted
- Can share between multiple containers
- Easier backup and migrate
- Best for production

Commands:
docker volume create myvolume = create
docker volume ls = list all
docker volume inspect myvolume = inspect (see mount path, driver, date)
docker volume rm myvolume = remove specific
docker volume prune = remove all unused

Using volume with container:
Syntax: docker run -v <volume_name>:<path_inside_container> image

Example: docker run -d -v myvolume:/app/data --name mycontainer ubuntu

Write data:
docker exec -it mycontainer sh -c "echo 'Hello Volume' > /app/data/test.txt"

Remove container:
docker rm -f mycontainer

Run NEW container with same volume - data still there:
docker run -it -v myvolume:/app/data ubuntu sh
cat /app/data/test.txt = Hello Volume

2. Bind Mounts
Bind mount links specific folder on host directly into container.

Key difference: you specify exact host path - Docker doesn't manage.

Why use?
- Great for development (live code editing without rebuild)
- Changes on host instantly visible inside
- Changes inside instantly visible on host

Syntax: docker run -v <host_path>:<container_path> image

Linux: docker run -d -v $(pwd)/html:/usr/share/nginx/html -p 8080:80 nginx

Windows: docker run -it -v C:\app\data:/data --name my_app ubuntu /bin/bash

Volumes vs Bind Mounts:
- Managed by Docker: volumes yes, bind mounts no
- Best for production: volumes yes, bind mounts no
- Host path dependency: volumes no, bind mounts yes
- Performance: both good
- Portability: volumes high, bind mounts low
- Backup/migrate: volumes easier, bind mounts manual

3. Copy-on-Write (CoW)
Docker images use layered filesystem (OverlayFS).

Top: Container writable layer (thin, unique per container)
Middle: App layer (read-only, shared)
Middle: Dependencies layer (read-only, shared)
Bottom: Base OS layer (read-only, shared)

How CoW works:
1. Container reads file: read from shared read-only layer (fast)
2. Container modifies file: Docker copies it up to writable layer, then modifies copy
3. Original image layers never touched
4. Multiple containers from same image = one copy of image layers shared in storage

View layers:
docker history nginx = see all layers
docker info | grep "Storage Driver" = check driver (usually overlay2 on Linux)

Example MySQL with volume:
docker volume create mysql_data = create volume
docker run -d --name mysql_db -e MYSQL_ROOT_PASSWORD=root123 -v mysql_data:/var/lib/mysql mysql:8

Data persists even if container deleted.
  -e MYSQL_ROOT_PASSWORD=root \
  -v mysql_data:/var/lib/mysql \
  mysql:8

# Even if you delete this container...
docker rm -f mysql_db

# ...data is safe — run a new container with same volume
docker run -d \
  --name mysql_db_new \
  -e MYSQL_ROOT_PASSWORD=root \
  -v mysql_data:/var/lib/mysql \
  mysql:8
```

---

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

 
