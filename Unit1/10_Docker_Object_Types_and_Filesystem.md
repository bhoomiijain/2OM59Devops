# Docker Object Types

**Four core Docker object types:**
- Image: read-only blueprint/template. Created by docker build or docker pull. Immutable. Multiple containers from one image.
- Container: running (or stopped) instance of image. Has writable layer. Isolated via namespaces. Resource-limited via cgroups. Ephemeral by default.
- Network: communication channel between containers. Three defaults: bridge, host, none. Controls container-to-container and container-to-outside communication.
- Volume: persistent storage outside container. Managed by Docker. Survives container deletion. Can share between containers.

**Image commands:**
docker images = list image
docker pull nginx = pull from registry
docker rmi nginx = remove
docker inspect nginx = metadata
docker history nginx = view layers

**Container commands:**
docker run -d --name myapp nginx = create + start
docker ps = list running
docker ps -a = list all
docker stop myapp = stop
docker start myapp = restart
docker rm myapp = delete
docker inspect myapp = detailed JSON
docker logs myapp = view output
docker logs -f myapp = follow logs live
docker stats myapp = CPU/memory usage
docker top myapp = running processes inside

Container interaction:
docker exec -it <id> bash = open terminal inside
docker attach <id> = attach to main process
docker cp <id>:/path ~/Desktop/ = copy from container
docker cp ~/Desktop/file <id>:/path = copy to container
docker kill <id> = force stop
docker container prune = remove all stopped

Network commands:
docker network ls = list networks
docker network create mynetwork = create custom
docker network inspect mynetwork = show details
docker network rm mynetwork = delete
docker network connect mynetwork myapp = add container to network
docker network disconnect mynetwork myapp = remove container

**Using network at run time:**
docker run -d --network mynetwork --name myapp nginx

**Volume commands:**
docker volume create myvolume = create
docker volume ls = list
docker volume inspect myvolume = details (mount path)
docker volume rm myvolume = delete
docker volume prune = delete unused

Using volume with container:
docker run -d -v myvolume:/app/data --name myapp ubuntu

Docker Layering & Filesystem:
Uses OverlayFS (Union Filesystem) to stack layers:
Top: Container writable layer (thin)
Middle layers: App, dependencies, runtime (read-only)
Bottom: Base OS layer (read-only)

Copy-on-Write (CoW):
- Containers share image layers (read-only)
- When file modified: Docker copies it up to writable layer
- Original image layers never touched
- Multiple containers from same image = one copy of layers in storage
- Efficient storage usage

View layers:
docker history nginx = see layers
docker info | grep "Storage Driver" = check driver (usually overlay2)

---

> 📅 *Notes from Class — Unit I*  
> 📖 *Reference: Slides + New PPT (Unit_2 slides 36–44)*
