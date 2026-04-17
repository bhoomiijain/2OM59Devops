# Container Interaction with Host

**docker exec:** run commands inside already running container
docker exec -it <id> bash = open interactive bash terminal
docker exec -it <id> sh = open sh (for Alpine images)
docker exec -it <id> ls / = run single command
docker exec -it <id> cat /etc/os-release = check OS
docker exec -it <id> sh -c "echo 'Hello' > /data/test.txt" = run multiple commands

Flags: -i = interactive (keep stdin), -t = allocate TTY, -d = detached (background)

docker attach: connects terminal to main process (PID 1) of container
docker attach <id>

Warning: Ctrl+C here stops container. Use Ctrl+P Ctrl+Q to detach safely.

Difference:
- docker exec: creates new process. Safe to exit with exit or Ctrl+D
- docker attach: joins main process. Need Ctrl+P Ctrl+Q to detach without stopping

docker cp: copy files both ways without entering container

FROM container → TO host:
docker cp <id>:/path/inside /path/on/host

FROM host → TO container:
docker cp /path/on/host <id>:/path/inside

Examples:
docker cp mycontainer:/app/app.log ~/Desktop/app.log = copy log from container to desktop
docker cp ~/Desktop/config.json mycontainer:/app/config.json = copy config into container
docker cp mycontainer:/data/test.txt C:\Users\HP\Desktop\ = Windows

Lab Exercise:

Step 1: Run container
docker run -d --name myubuntu ubuntu sleep 300

Step 2: Enter it
docker exec -it myubuntu /bin/bash

Step 3: Create files inside
mkdir /data
echo "Hello Docker" > /data/test.txt
exit

Step 4: Attach and detach safely
docker attach myubuntu
Press: Ctrl+P then Ctrl+Q

Step 5: Copy file from container to host
docker cp myubuntu:/data/test.txt ~/Desktop/

Step 6: Copy file from host to container
docker cp ~/Desktop/test.txt myubuntu:/data/sample.txt

Step 7: Verify both files inside container
docker exec -it myubuntu ls /data
docker exec -it myubuntu cat /data/sample.txt

Quick Reference:
docker exec -it <id> bash = enter container
docker exec -it <id> sh -c "command" = run command
docker logs -f <id> = follow logs
docker attach <id> = attach to main process
docker cp <id>:/src /dst = copy file out
docker cp /src <id>:/dst = copy file in

# Copy container → host
docker cp <id>:/container/path /host/path

# Copy host → container
docker cp /host/path <id>:/container/path

# Live resource usage
docker stats <id>

# Running processes inside
docker top <id>

# Full config in JSON
docker inspect <id>
```

---

> 📅 *Notes from Class — Unit II*  
> 📖 *Reference: New PPT slides 39–44 (Unit_2__1_ + Basic_Docker_commands)*
