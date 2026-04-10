//index.html
<!DOCTYPE html>
<html>
<head>
    <title>Medium Level Docker App</title>
</head>
<body>
    <h1>Welcome to My Custom Docker Nginx App</h1>
    <p>This is a medium-level Docker project.</p>
</body>
</html>

notepad default.conf
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}

notepad Dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
COPY default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
------

or 

Create app.js

const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Docker Node App Running!");
});

app.listen(3000, "0.0.0.0", () => {
  console.log("Server running on port 3000");
});

Create package.json
{
  "name": "node-docker-app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.2"
  }
}

Create Dockerfile 

FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]

-------

rename-Item old new or ren
docker build -t node-demo:v1 .
Docker run -d -p 3000:3000 --name node-container node-demo:v1
docker ps
OPEN BROWSER
http://localhost:3000
Docker Node App Running!

curl http://localhost