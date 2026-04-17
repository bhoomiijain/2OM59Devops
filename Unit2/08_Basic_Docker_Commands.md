# Basic Docker Commands (INT332)

## Docker Version & System Information

1. **docker --version** : Shows installed Docker version and build number.
   - Example output: Docker version 24.0.2, build abcdef123

2. **docker info** : Get detailed Docker daemon and system information.
   - Total containers running/paused/stopped
   - Total images available
   - Storage driver used (overlay2 is most common)
   - Docker root directory (/var/lib/docker)
   - Server version
   - Kernel version
   - Operating system

## Image Management Commands

1. **docker pull image:tag** : Download image from Docker Hub or specified registry.
   - If tag omitted, defaults to 'latest'
   - Examples:
     - docker pull ubuntu  (pulls ubuntu:latest)
     - docker pull nginx:1.25 (pulls specific version)
     - docker pull python:3.11-slim (pulls variant)
     - docker pull mysql:8
     - docker pull ghcr.io/bhoomiijain/myapp:v1 (from GHCR)

2. **docker images** : List all images available locally.
   - Shows: Repository, Tag, Image ID, Size, Created Date
   - Output columns:
     - REPOSITORY: image name
     - TAG: version identifier
     - IMAGE ID: unique identifier (first 12 chars shown)
     - SIZE: image size on disk
     - CREATED: when image was created

3. **docker history image:tag** : Show all layers in image (each Dockerfile line = one layer).
   - Each layer shows: Command that created it, Size of layer, Creation date
   - Example: docker history nginx:alpine
   - Useful: understand what's inside image, debug large images

4. **docker inspect image:tag** : Get complete metadata about image in JSON format.
   - Includes: architecture, OS, environment variables, exposed ports, volumes, working directory, default command
   - Example: docker inspect ubuntu:22.04
   - Useful: understand image configuration before running

## Container Execution Commands

**docker run [OPTIONS] IMAGE [COMMAND] [ARG]** : Create and start a new container from image.

Syntax: `docker run <options> <image> <command> <args>`

Common Options:
- **-it** : Interactive terminal mode (attach STDIN and TTY even if not attached)
- **-d** : Detached mode (run in background, return container ID)
- **--name** : Assign a name to container
- **--rm** : Automatically remove container when it exits
- **-p** : Map ports (host_port:container_port)
- **-e** : Set environment variable
- **-v** : Mount volume or bind mount
- **-w** : Set working directory inside container
- **--cpus** : Limit CPU cores (e.g., 1.5)
- **-m** : Limit memory (e.g., 512m)
- **--restart** : Restart policy (no, always, unless-stopped, on-failure)

Examples:

1. **docker run ubuntu echo "Hello Docker"** : Creates container, runs echo command, exits.
   - Container removed automatically (with --rm option)

2. **docker run -it ubuntu bash** : Creates container, opens interactive bash shell.
   - You can type commands inside
   - Type `exit` to quit the container

3. **docker run -d --name webserver nginx** : Creates container named 'webserver', runs in background.
   - Returns container ID, doesn't occupy terminal

4. **docker run -d --name webserver -p 8080:80 nginx** : Port mapping.
   - Access nginx at http://localhost:8080 (host) → port 80 (container)

5. **docker run -it -e MY_VAR=hello ubuntu bash** : Set environment variable inside container.
   - Inside container: echo $MY_VAR → outputs 'hello'

6. **docker run -it -e APP_ENV=production -e DEBUG=false nginx bash** : Multiple environment variables.

7. **docker run -d --name mysql_db -e MYSQL_ROOT_PASSWORD=root123 -e MYSQL_DATABASE=mydb mysql:8** : Start MySQL with required environment variables.

8. **docker run -d --name app -v /data ubuntu** : Attach volume /data inside container.
   - Docker creates default volume

9. **docker run -d --name app -v myvolume:/app/data ubuntu** : Attach named volume 'myvolume' to /app/data.

10. **docker run -it --name app -v $(pwd):/app ubuntu bash** : Bind mount current host directory to /app.
    - Use case: development

11. **docker run -d --name app --cpus 2 -m 512m ubuntu** : Limit container to 2 CPU cores and 512MB RAM.

12. **docker run --rm -it ubuntu bash** : Automatically remove container when it exits.
    - Use case: cleanup

## Listing & Inspecting Containers

1. **docker ps** : List all currently running containers.
   - Shows: Container ID, Image, Status, Ports, Names
   - Example output:
     ```
     CONTAINER ID   IMAGE  COMMAND        STATUS         PORTS               NAMES
     abc123def456   nginx  "nginx -g..."  Up 5 minutes    0.0.0.0:8080->80    webserver
     ```

2. **docker ps -a** : List all containers (running + stopped + exited).
   - Useful: see history of all containers created

3. **docker ps -aq** : List only container IDs (quiet mode).
   - Useful with scripts

4. **docker inspect CONTAINER_ID/NAME** : Get detailed JSON info about container.
   - Current state (running, paused, stopped)
   - Process ID
   - Ports mapped
   - Volumes attached
   - Environment variables
   - Mounts (volumes/binds)
   - Network settings
   - IP address
   - Example: docker inspect webserver

5. **docker logs CONTAINER_ID/NAME** : View output of container (stdout).
   - Shows: console output, application logs, errors
   - Example: docker logs webserver

6. **docker logs -f CONTAINER_ID/NAME** : Follow logs in real-time (-f = follow).
   - Useful: monitor running application
   - Press Ctrl+C to stop following

7. **docker logs --tail 50 CONTAINER_ID/NAME** : Show last 50 lines of logs.

8. **docker stats CONTAINER_ID/NAME** : Show live resource usage: CPU, memory, network I/O, block I/O.
   - Updates every second
   - Press Ctrl+C to exit

## Container Control Commands

1. **docker stop CONTAINER_ID/NAME** : Stop container gracefully (SIGTERM signal).
   - Gives container time to cleanup (~10 seconds)

2. **docker kill CONTAINER_ID/NAME** : Force stop container immediately (SIGKILL signal).
   - No cleanup time

3. **docker restart CONTAINER_ID/NAME** : Stop and restart container.
   - Useful: when container misbehaves

4. **docker pause CONTAINER_ID/NAME** : Pause container (freeze all processes).
   - Useful: temporary pause without removing

5. **docker unpause CONTAINER_ID/NAME** : Resume paused container.

6. **docker start CONTAINER_ID/NAME** : Start a stopped container.
   - Container resumes with same configuration

## Container Interaction Commands

1. **docker exec -it CONTAINER_ID/NAME COMMAND** : Execute command inside running container.
   - Opens interactive bash shell inside container
   - You're now inside container
   - Examples:
     - docker exec -it webserver bash (interactive shell)
     - docker exec -it webserver ls / (list root directory)
     - docker exec -it mysql_db mysql -u root -p (access MySQL)
     - docker exec webserver cat /var/log/nginx/access.log (view nginx logs)

2. **docker attach CONTAINER_ID/NAME** : Attach to main process of running container.
   - Different from exec: exec runs new process, attach connects to existing process

3. **docker cp SOURCE DEST** : Copy files between container and host.
   - From container to host: docker cp webserver:/var/log/nginx/access.log ./logs/
   - From host to container: docker cp ./index.html webserver:/usr/share/nginx/html/
   - More examples:
     - docker cp webserver:/etc/nginx/nginx.conf ./backup/
     - docker cp ./app.js myapp:/app/
     - docker cp webserver:/var/www/html/ ./website_backup/

## Container Cleanup

1. **docker rm CONTAINER_ID/NAME** : Remove a stopped container.
   - Error if container still running (use docker stop first)

2. **docker rm -f CONTAINER_ID/NAME** : Force remove even if running.
   - Kills first, then removes

3. **docker rm $(docker ps -aq)** : Remove all stopped containers.
   - Warning: dangerous operation!

4. **docker container prune** : Remove all stopped containers.
   - Asks for confirmation

5. **docker ps -a --filter status=exited** : Show only exited containers.
   - Useful before cleanup

## Image Cleanup

1. **docker rmi IMAGE_ID/NAME:TAG** : Remove image locally.
   - Error if image has running containers

2. **docker rmi -f IMAGE_ID/NAME:TAG** : Force remove image.
   - Even if running containers use it

3. **docker image ls** : List all images.
   - Same as docker images

4. **docker image rm IMAGE_NAME** : Remove image by name.

5. **docker image prune** : Remove unused images.
   - Images not used by any container

6. **docker rmi $(docker images -q)** : Remove all images.
   - Warning: dangerous operation! use with caution

## Echo Command Inside Containers

Echo writes text to stdout or file.

1. **docker exec -it ubuntu bash -c "echo Hello Docker"** : Output text.
   - Outputs: Hello Docker

2. **docker exec -it ubuntu sh -c "echo 'My content' > /tmp/file.txt"** : Creating files with echo.

3. **docker exec -it ubuntu sh -c "echo 'Additional line' >> /tmp/file.txt"** : Appending to files.

4. **docker exec -it ubuntu cat /tmp/file.txt** : Viewing created files.

## Practical Command Chains

1. **docker run -it ubuntu bash -c "echo 'First line' > /data/test.txt && cat /data/test.txt"** : Execute multiple commands in one go.

2. **docker exec -it myapp sh -c "ps aux | grep node"** : Pipe commands inside container.

## Real World Examples

**Example 1: Deploy nginx web server**
```
docker run -d --name mywebsite -p 8080:80 nginx
docker exec -it mywebsite bash -c "echo '<h1>My Site</h1>' > /usr/share/nginx/html/index.html"
curl http://localhost:8080
```

**Example 2: Run MySQL**
```
docker run -d --name mysql_prod -p 3306:3306 -e MYSQL_ROOT_PASSWORD=secure123 -v mysql_data:/var/lib/mysql mysql:8
docker exec -it mysql_prod mysql -u root -p
```

**Example 3: Python application**
```
docker run -it python:3.11 python -c "print('Hello from Python')"
```

**Example 4: Monitor container**
```
docker run -d --name app myimage
docker logs -f app
docker stats app
```

## Command History

1. **docker history ubuntu:22.04** : Shows layers in Ubuntu image.

2. **docker history nginx:alpine** : Shows layers in Nginx image.

## Exit Codes

When container exits, check exit status:
- Command: **docker ps -a** (STATUS column shows exit code)
- Exit code 0 = success
- Exit code 1+ = error occurred