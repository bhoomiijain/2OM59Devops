# Unit 2 Docker Practical Tasks (INT332)

TASK 1: Basic Docker Commands and Image Pulling

Objective: Master docker version, info, history, pull commands

Questions:
1. What is your installed Docker version and build number?
2. How many containers are currently running on your system?
3. What storage driver does your Docker use?
4. Pull Ubuntu 22.04 image. What is the image ID?
5. View the layers of nginx:alpine image using history command.

Commands to Execute:

docker --version
docker info
docker pull ubuntu:22.04
docker images
docker history nginx:alpine
docker inspect ubuntu:22.04

Expected Outcomes:
- Know your Docker version
- Understand Docker daemon info structure
- Image successfully pulled
- Can view image layers
- Understand image metadata

---

TASK 2: Running Containers with Basic Options

Objective: Learn docker run with various options

1. Run Ubuntu container in interactive mode and type commands inside:

docker run -it ubuntu bash
Inside container:
echo "Hello Docker"
ls /
exit

Verify: container stopped

2. Run Nginx in detached mode:

docker run -d --name webserver nginx

Verify: docker ps (showing running container)

3. Run with custom name:

docker run -d --name my-app nginx

Verify: docker ps (showing custom name)

4. Run with automatic cleanup (--rm):

docker run --rm -it ubuntu bash
Inside: echo "This container will auto-delete"
exit

Verify: docker ps -a (container not in history)

Expected Outcomes:
- Understand interactive vs detached modes
- Custom naming containers
- Container lifecycle
- Auto-cleanup mechanism

---

TASK 3: Port Mapping (-p flag)

Objective: Expose container ports to host

1. Run Nginx without port mapping:

docker run -d --name nginx1 nginx

Try accessing: http://localhost:80
Result: Connection refused

2. Run Nginx WITH port mapping:

docker run -d -p 8080:80 --name nginx2 nginx

Try accessing: http://localhost:8080
Result: Nginx home page loads successfully!

3. Port mapping multiple ports:

docker run -d -p 8080:80 -p 8443:443 --name nginx3 nginx

Verify: docker ps (see port mappings)

4. Map different host port:

docker run -d -p 9000:80 --name nginx4 nginx

Access: http://localhost:9000

Why does port mapping matter?
Answer: Containers isolated. Without mapping, port inaccessible from host.

Expected Outcomes:
- Understand containerport vs host port
- Know when port mapping required
- Multiple port mappings
- Browser access to containerized services

---

TASK 4: Environment Variables (-e flag)

Objective: Set and use environment variables

1. Simple echo with env var:

docker run -it -e MY_NAME=Bhoomi ubuntu bash
Inside: 
echo $MY_NAME
echo "Hello $MY_NAME"
exit

2. With vs Without env vars:

WITH:
docker run -it -e COLLEGE=CSE ubuntu bash
Inside: echo $COLLEGE → outputs: CSE

WITHOUT:
docker run -it ubuntu bash
Inside: echo $COLLEGE → outputs: (empty)

3. Multiple environment variables:

docker run -d \
  -e APP_ENV=production \
  -e LOG_LEVEL=info \
  -e DEBUG=false \
  nginx

Verify: docker exec <container_id> env

4. MySQL with required env vars (fails without):

Without env vars:
docker run -d mysql:8
docker ps -a (see ERROR status)

With env vars (succeeds):
docker run -d \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=college \
  mysql:8

Verify: docker ps (see RUNNING status)

Expected Outcomes:
- Understand environment variables purpose
- Set variables at runtime
- Difference between with/without env vars
- Multiple environment variable configurations
- Know which services need env vars to function

---

TASK 5: Container Interaction and File Operations

Objective: Execute commands in containers, copy files

1. Execute command inside running container:

docker run -d --name myapp ubuntu sleep 1000

docker exec myapp ls /
docker exec myapp pwd
docker exec myapp cat /etc/os-release

2. Open interactive shell inside container:

docker exec -it myapp bash
(You're inside container now)
mkdir /data
echo "Hello Docker" > /data/test.txt
cat /data/test.txt
exit

3. Copy file FROM container TO host:

docker cp myapp:/data/test.txt ~/Desktop/

Verify: file appears on Desktop

4. Copy file FROM host TO container:

echo "New data" > ~/Desktop/new.txt
docker cp ~/Desktop/new.txt myapp:/data/

docker exec -it myapp cat /data/new.txt

Expected Outcomes:
- Execute commands inside running containers
- Interactive shell access
- File transfer container ↔ host
- Understand docker cp usage

---

TASK 6: Container Lifecycle Management

Objective: Start, stop, restart, remove containers

1. Create and run container:

docker run -d --name server nginx

2. Stop container (graceful):

docker stop server
docker ps (container not running)

3. Start stopped container:

docker start server
docker ps (container running again)

4. Restart container:

docker restart server

5. Kill container (force stop):

docker kill server

6. Remove stopped container:

docker rm server

7. Force remove running container:

docker rm -f <container_id>

Remove all stopped containers:
docker container prune

Expected Outcomes:
- Container state transitions
- Graceful vs forced termination
- Container restart
- Efficient cleanup

---

TASK 7: Container Listing and Inspection

Objective: View container status and details

1. List running containers:

docker ps

2. List ALL containers:

docker ps -a

3. Get detailed JSON info:

docker inspect <container_id>
(Shows: UUID, state, mounts, network, env vars, etc)

4. View container logs:

docker run -d --name logger nginx

docker logs logger
docker logs -f logger (follow real-time)
Ctrl+C to stop

5. View resource usage:

docker stats <container_name>
(CPU, memory, I/O, network stats)

Expected Outcomes:
- Understand container listing
- Interpret container states
- Debug using logs
- Monitor resource usage

---

TASK 8: Image Management

Objective: Build, tag, push, remove images

1. View all images:

docker images

2. View image history (layers):

docker history ubuntu:22.04

3. Inspect image metadata:

docker inspect nginx:alpine

4. Remove image (if no containers using it):

docker rmi nginx

5. Force remove image:

docker rmi -f <image_id>

6. Clean up unused images:

docker image prune

Expected Outcomes:
- Manage local images
- Understand image layers
- Image metadata
- Image cleanup

---

TASK 9: Dockerfile and Image Creation

Objective: Create custom image from Dockerfile

Step 1: Create project directory:

mkdir my-docker-app
cd my-docker-app

Step 2: Create Dockerfile:

cat > Dockerfile << 'EOF'
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y curl
RUN mkdir -p /app
WORKDIR /app
COPY hello.txt /app/
ENV APP_NAME=MyApp
CMD ["cat", "hello.txt"]
EOF

Step 3: Create file to copy:

echo "Hello from My Custom Docker Image!" > hello.txt

Step 4: Build image:

docker build -t my-custom-app:v1 .

Step 5: Run container from custom image:

docker run my-custom-app:v1

Output: Hello from My Custom Docker Image!

Step 6: Verify image created:

docker images | grep my-custom-app

Expected Outcomes:
- Write Dockerfile
- Build custom image
- Run container from custom image
- Understand build process

---

TASK 10: Multi-Container Application with Networking

Objective: Create networked multi-container setup

Step 1: Create custom network:

docker network create app-net

Step 2: Run MySQL on network:

docker run -d \
  --name mysql_db \
  --network app-net \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=testdb \
  mysql:8

Step 3: Run another container on same network:

docker run -it \
  --name app_container \
  --network app-net \
  ubuntu bash

Inside container:
ping mysql_db
(Should work! DNS resolves container name)
exit

Step 4: Verify network:

docker network inspect app-net
(Shows both containers connected)

Expected Outcomes:
- Create custom networks
- Multi-container communication
- DNS resolution by name
- Network isolation

---

TASK 11: Working with Volumes

Objective: Create and use persistent volumes

Step 1: Create volume:

docker volume create my_data

Step 2: Run container with volume:

docker run -d \
  --name app_with_volume \
  -v my_data:/data \
  ubuntu \
  sleep 1000

Step 3: Create file inside container:

docker exec app_with_volume bash -c "echo 'Persistent Data' > /data/file.txt"

Step 4: Verify file exists:

docker exec app_with_volume cat /data/file.txt

Step 5: Stop and remove container:

docker stop app_with_volume
docker rm app_with_volume

Step 6: Run NEW container with SAME volume:

docker run -d \
  --name app_with_volume_2 \
  -v my_data:/data \
  ubuntu \
  sleep 1000

Step 7: Verify file persisted:

docker exec app_with_volume_2 cat /data/file.txt
(Same content! Data persisted!)

Expected Outcomes:
- Create volumes
- Attach volumes to containers
- Understand volume persistence
- Data survives container deletion

---

TASK 12: Bind Mounts for Development

Objective: Use bind mounts for live code editing

Step 1: Create project directory:

mkdir dev-app
cd dev-app
echo "console.log('Version 1');" > app.js

Step 2: Run container with bind mount:

docker run -d \
  --name dev_container \
  -v $(pwd):/app \
  -w /app \
  node:18 \
  node app.js

Step 3: Edit file on host (while container running):

echo "console.log('Version 2 - Updated!');" > app.js

Step 4: Observe container output:

docker logs dev_container
(Shows updated code output immediately!)

Expected Outcomes:
- Understand bind mounts
- Live code editing
- No rebuild necessary
- Development workflow efficiency

---

TASK 13: Database Container with Environment Variables and Port Mapping

Objective: Deploy realistic database container

Run MySQL:

docker run -d \
  --name production_db \
  -p 3307:3306 \
  -e MYSQL_ROOT_PASSWORD=secure_pass123 \
  -e MYSQL_DATABASE=company_db \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=app_pass \
  mysql:8

Verify running:

docker ps (confirm port mapping 3307:3306)

Connect from host:

mysql -h 127.0.0.1 -P 3307 -u root -p
(Password: secure_pass123)

Inside MySQL:
SHOW DATABASES;
USE company_db;
CREATE TABLE users (id INT, name VARCHAR(100));
INSERT INTO users VALUES (1, 'John');
SELECT * FROM users;
exit

Verify persistence:

docker logs production_db

Expected Outcomes:
- Understand database container deployment
- Port mapping for external access
- Environment variable configuration
- Database connection verification

---

TASK 14: Complete Full-Stack Application

Objective: Deploy multi-layer application

Step 1: Create custom network:

docker network create fullstack

Step 2: Deploy database:

docker run -d \
  --name db \
  --network fullstack \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=appdb \
  mysql:8

Step 3: Create application (optional):

docker run -d \
  --name backend \
  --network fullstack \
  -e DB_HOST=db \
  -e DB_USER=root \
  -e DB_PASS=root \
  node:18

Step 4: Deploy web frontend:

docker run -d \
  --name frontend \
  --network fullstack \
  -p 8080:80 \
  -e API_HOST=backend \
  nginx

Step 5: Verify all running:

docker ps (all 3 containers running)
docker network inspect fullstack (all 3 connected)

Step 6: Test connectivity:

docker exec backend ping db (success!)
docker exec frontend ping backend (success!)

Access web: http://localhost:8080

Expected Outcomes:
- Multi-container orchestration
- Network communication
- Environment configuration
- Full-stack deployment