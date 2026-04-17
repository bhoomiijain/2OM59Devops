# Docker Projects

Project 1: Custom Nginx App

Files: index.html, default.conf, Dockerfile

index.html:
<!DOCTYPE html>
<html>
<head>
    <title>Docker App</title>
</head>
<body>
    <h1>Welcome to My Custom Docker Nginx App</h1>
</body>
</html>

default.conf:
server {
    listen 80;
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}

Dockerfile:
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
COPY default.conf /etc/nginx/conf.d/default.conf
EXPOSE 80

Commands:
docker build -t custom-nginx:v1 .
docker run -d -p 8080:80 --name nginx-container custom-nginx:v1
Open browser: http://localhost:8080

Project 2: Node.js Express App

Files: app.js, package.json, Dockerfile

app.js:
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Docker Node App Running!");
});

app.listen(3000, "0.0.0.0", () => {
  console.log("Server running on port 3000");
});

package.json:
{
  "name": "node-docker-app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.2"
  }
}

Dockerfile:
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]

Commands:
docker build -t node-demo:v1 .
docker run -d -p 3000:3000 --name node-container node-demo:v1
docker ps
Open browser: http://localhost:3000

Rename files:
rename-Item old new (PowerShell)
ren old new (older Windows)