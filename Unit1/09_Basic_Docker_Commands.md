# Basic Docker Commands

Image Commands:
docker --version = check Docker version
docker info = detailed Docker info
docker pull ubuntu = download image
docker images = list downloaded images
docker history httpd = view image history
docker rmi ubuntu = delete image
docker rmi -f <id> = force delete

Container Lifecycle:
docker run ubuntu = create + run container
docker run -it ubuntu /bin/bash = run with interactive terminal
docker run -d nginx = run background (detached)
docker run -d -p 8080:80 --name mynginx nginx = run with name + port
docker run --rm ubuntu echo "Hello" = auto-remove after stop
docker ps = list running containers
docker ps -a = list all containers (running + stopped)
docker start <id> = start stopped container
docker stop <id> = stop running container
docker restart <id> = restart container
docker pause <id> = pause container
docker rm <id> = remove container
docker rm -f <id> = force remove even if running

Container Interaction:
docker exec -it <id> bash = open bash terminal inside container
docker logs <id> = show container output
docker inspect <id> = detailed config in JSON
docker stats = live CPU + memory usage
docker top <id> = running processes inside

Port Mapping flag -p:
-p <HOST_PORT>:<CONTAINER_PORT>
docker run -d -p 8080:80 --name mynginx nginx
Maps host port 8080 to container port 80
Access: localhost:8080 in browser

docker run Flags Summary:
-d = detached (background)
-it = interactive + tty
--name = give container name
--rm = auto-remove on exit
-p = port mapping, -e = environment variable, -v = volume mount

Volume Commands:
docker volume create myvolume = create volume
docker volume ls = list all volumes
docker volume inspect myvolume = show details
docker volume rm myvolume = delete

Network Commands:
docker network ls = list all networks
docker network create mynetwork = create custom network
docker network inspect mynetwork = inspect network
docker network rm mynetwork = remove network

Cleanup:
docker stop $(docker ps -aq) = stop all running containers
docker rm $(docker ps -aq) = remove all containers
docker rmi $(docker images -q) = delete all images
docker system prune -a = clean unused containers/images/networks
docker system prune -a --volumes = clean everything including volumes

Examples:

Nginx with port mapping:
docker pull nginx
docker run -d -p 8080:80 --name mynginx nginx
docker ps
docker stop mynginx
docker start mynginx
docker rm -f mynginx

Ubuntu interactive:
docker run -it ubuntu bash
Inside: echo "Hello from container"
Inside: exit

Apache (httpd):
docker pull httpd
docker run -d --name my-apache -p 8080:80 httpd
docker exec -it my-apache bash -c "echo '<h1>Hello!</h1>' > /usr/local/apache2/htdocs/index.html"
curl http://localhost:8080

MySQL with env variables:
docker run -d -p 3307:3306 --name mysql-test -e MYSQL_ROOT_PASSWORD=root123 -e MYSQL_DATABASE=college mysql:8
docker exec -it mysql-test bash
mysql -h 127.0.0.1 -u root -p


