# Environment Variables in Docker

What are Environment Variables?

Environment variables = key-value pairs that provide configuration data to programs.

Format: KEY=VALUE

Examples:
DATABASE_URL=localhost:5432
API_KEY=secret123
APP_ENV=production
DEBUG=true
LOG_LEVEL=info

Analogy: Like putting sticky notes on a desk. Program reads notes and changes behavior.

Why Use Environment Variables?

1. Configuration Without Rebuilding Image

Without env vars:
- Want different database? Edit code → rebuild image → deploy
- Slow and error-prone

With env vars:
- Change value at runtime: docker run -e DB_HOST=db.example.com
- Same image, different behavior instantly

2. Secrets Management

Never hardcode passwords/API keys in Docker image!

Bad (DON'T DO THIS):
Dockerfile:
RUN echo "DB_PASS=mysecret" > /app/.env
Problem: Password visible in image layers forever.

Good (DO THIS):
docker run -e DB_PASSWORD=mysecret myapp
Secret only in memory, not in image.

3. Multiple Environments (Dev/Staging/Prod)

Same image used in three environments:
docker run -e APP_ENV=development myapp (developers)
docker run -e APP_ENV=staging myapp (testing)
docker run -e APP_ENV=production myapp (users)

Same image, different logs, debug levels, database connections.

4. Dynamic Configuration

Change container behavior without rebuilding:
docker run -e WORKERS=5 myapp (5 worker threads)
docker run -e WORKERS=10 myapp (10 worker threads)

## Setting Environment Variables: Methods

**Method 1: Docker run with -e flag**

```bash
docker run -e VARIABLE=value image:tag
```

Examples:
```bash
docker run -e APP_MODE=development ubuntu bash
docker run -e DB_HOST=localhost -e DB_PORT=5432 myapp
```

Inside container:
```bash
echo $APP_MODE          # outputs: development
echo $DB_HOST           # outputs: localhost
```

**Method 2: Multiple environment variables**

```bash
docker run -e VAR1=val1 -e VAR2=val2 -e VAR3=val3 image:tag
```

Example:
```bash
docker run -d \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=college \
  -e MYSQL_USER=admin \
  -e MYSQL_PASSWORD=admin123 \
  mysql:8
```

Note: Without these vars, MySQL fails to start (required configuration).

**Method 3: Pass from host system**

On host:
```bash
export APP_PORT=8080
```

In docker run:
```bash
docker run -e APP_PORT myapp  # Container sees: APP_PORT=8080
```

**Method 4: Using --env-file**

Create .env file:
```
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=secret123
APP_PORT=8080
```

Run container:
```bash
docker run -d --env-file .env myapp
# All variables from .env file loaded into container
```

**Method 5: Dockerfile ENV instruction**

```dockerfile
FROM python:3.11
ENV NODE_ENV=production
ENV APP_DEBUG=false
COPY app.py /app/
CMD ["python", "/app/app.py"]
```

When container runs, these vars available.
DB_PASS=secret
APP_ENV=production

Run container:
docker run --env-file .env myapp

All variables from .env file set in container.

Method 5: Dockerfile ENV instruction

Dockerfile:
FROM ubuntu:22.04
ENV APP_NAME=myapp
ENV VERSION=1.0

Command to view:
docker run image env

Container starts with APP_NAME and VERSION variables.

Environment Variables Examples:

Example 1: Simple echo

docker run -it -e MY_NAME=Bhoomi ubuntu bash

Inside container:
echo $MY_NAME → outputs: Bhoomi
echo "Hello $MY_NAME" → outputs: Hello Bhoomi

Example 2: Without environment variable

docker run -it ubuntu bash

Inside container:
echo $MY_NAME → outputs: (nothing, variable not set)

Difference (with vs without):
WITH: docker run -e COLLEGE=CSE ubuntu bash
Inside: echo $COLLEGE → outputs: CSE

WITHOUT: docker run ubuntu bash
Inside: echo $COLLEGE → outputs: (empty)

Example 3: Database configuration

docker run -d \
  -e DB_HOST=db.company.com \
  -e DB_USER=admin \
  -e DB_PASSWORD=secret123 \
  -e DB_NAME=production_db \
  myapp

Application reads env vars, connects to database.

Example 4: Multiple services with different configs

Service 1 (Development):
docker run -d -e ENV=dev -e LOG_LEVEL=debug service1

Service 2 (Production):
docker run -d -e ENV=prod -e LOG_LEVEL=error service2

Same container image, different behaviors.

Port Mapping:

Port mapping: HOST_PORT:CONTAINER_PORT

Syntax:
docker run -p <host_port>:<container_port> image:tag

Why needed?
Containers run in isolated network.
Applications inside not accessible from host/browser by default.

Without -p:
docker run nginx
Try accessing http://localhost:80 → Connection refused (nginx inside container, not exposed)

With -p:
docker run -p 8080:80 nginx
Access http://localhost:8080 → Works! (mapped to container port 80)

Port mapping process:
Browser → localhost:8080 → Docker Host → Container:80 → Nginx

Examples:

Web server mapping:
docker run -d -p 8080:80 nginx
Access: http://localhost:8080

Multiple ports:
docker run -d -p 8080:80 -p 443:443 nginx
HTTP: http://localhost:8080
HTTPS: https://localhost:443

Database port:
docker run -d -p 3307:3306 mysql:8
Connect from host: mysql -h 127.0.0.1 -P 3307 -u root -p

Application port:
docker run -d -p 5000:8000 myapp
App listens on port 8000, accessed via localhost:5000

Real Examples:

Example 1: Nginx with port mapping

docker run -d --name webserver -p 8080:80 nginx

Verify:
docker ps
curl http://localhost:8080

Example 2: MySQL with environment variables

docker run -d \
  --name mysql_test \
  -p 3307:3306 \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=testdb \
  mysql:8

Verify:
docker ps
mysql -h 127.0.0.1 -P 3307 -u root -p (password: root123)

Example 3: Node application

docker run -d \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e API_KEY=abc123 \
  -v $(pwd):/app \
  node:18 \
  node /app/server.js

Example 4: Python Flask app

docker run -d \
  -p 5000:5000 \
  -e FLASK_ENV=development \
  -e DEBUG=true \
  python-app:v1

Common Environment Variable Patterns:

Application Environment:
-e APP_ENV=development
-e APP_ENV=staging
-e APP_ENV=production

Database Configuration:
-e DB_HOST=localhost
-e DB_PORT=5432
-e DB_USER=admin
-e DB_PASSWORD=secret
-e DB_NAME=mydb

API Configuration:
-e API_KEY=abc123
-e API_SECRET=xyz789
-e API_URL=https://api.example.com

Logging Configuration:
-e LOG_LEVEL=debug
-e LOG_LEVEL=info
-e LOG_LEVEL=error

Feature Flags:
-e FEATURE_A=enabled
-e FEATURE_B=disabled

Server Configuration:
-e PORT=8000
-e WORKERS=4
-e TIMEOUT=30

Verification:

View environment variables inside container:

docker run -it ubuntu bash
Inside: env (shows all variables)
Inside: env | grep MY_VAR (find specific variable)

Or without entering:

docker exec container_id env (view all env vars)
docker exec container_id printenv DB_HOST (view specific)

Limits:
Env var limits: practically unlimited
Container limit per variable: ~128KB
Best practice: keep values reasonable size

Don't know config at image build time? Set at run time.
Scenario: Need different port per deployment.

docker run -e PORT=3000 myapp (runs on 3000)
docker run -e PORT=4000 myapp (runs on 4000)

Same image, different ports.

How Env Variables Work:

Inside container: program reads environment variables using programming language.

Node.js:
const dbHost = process.env.DB_HOST;
const dbPort = process.env.DB_PORT || 5432; (default if not set)

Python:
import os
db_host = os.getenv('DB_HOST')
db_port = os.getenv('DB_PORT', 5432)

Java:
String dbHost = System.getenv("DB_HOST");

Shell/Bash:
echo $DB_HOST
echo $APP_ENV

Env vars only inside container - host doesn't see them.

Without Environment Variables:

Hardcoding Example:

app.js (bad practice):
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost:27017/mydb');

Problem:
- If database moves to 192.168.1.100, must edit code + rebuild
- If password changes, must edit code + rebuild
- Secret in image (visible to others who can inspect)
- Can't reuse image for different deployments
- Rebuild for each environment (dev/staging/prod)

With Environment Variables:

app.js (good practice):
const mongoose = require('mongoose');
const uri = process.env.MONGODB_URI || 'mongodb://localhost:27017/mydb';
mongoose.connect(uri);

Run with different values:
docker run -e MONGODB_URI=mongodb://user:pass@prod-db:27017/mydb myapp

Same code, different database.

Using -e Flag with docker run:

Single Environment Variable:

docker run -e KEY=VALUE image

Example:
docker run -e MY_VAR=hello ubuntu echo $MY_VAR
Output: hello (inside container)

Host doesn't see MY_VAR. Container-only.

With Named Container:
docker run -it -e MY_NAME=Bhoomi ubuntu bash
Inside container: echo $MY_NAME
Output: Bhoomi

Multiple Environment Variables:

docker run -e VAR1=value1 -e VAR2=value2 -e VAR3=value3 image

Example - MySQL:
docker run -d \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=college \
  -e MYSQL_USER=admin \
  -e MYSQL_PASSWORD=admin123 \
  mysql:8

Four variables set. MySQL container creates database and user automatically.

Example - MongoDB:
docker run -d \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=password123 \
  -e MONGO_INITDB_DATABASE=myapp_db \
  mongo

Example - Node.js App:
docker run -d \
  -e PORT=3000 \
  -e NODE_ENV=production \
  -e API_KEY=abc123xyz \
  -e DB_CONNECTION=mongodb://mongo:27017/mydb \
  -p 3000:3000 \
  node-app:v1

Checking Environment Variables Inside Container:

View all env variables:
docker exec -it <container_name> env

Shows all KEY=VALUE pairs.

Check specific variable:
docker exec -it <container_name> echo $VAR_NAME

Example:
docker exec -it webserver echo $PORT

Using Env Variables in Different Images:

Nginx (Web Server):

docker run -d \
  -e SERVER_NAME=example.com \
  -p 80:80 \
  nginx

Note: Need custom Nginx config to use variables (default Nginx doesn't use them).

Python Application:

docker run -d \
  -e FLASK_ENV=production \
  -e DEBUG=false \
  -p 5000:5000 \
  python-app:v1

Java Application:

docker run -d \
  -e JAVA_OPTS=-Xmx512m \
  -e APP_CONFIG=/app/config.properties \
  -p 8080:8080 \
  java-app:v1

PostgreSQL Database:

docker run -d \
  -e POSTGRES_USER=dbuser \
  -e POSTGRES_PASSWORD=securepass \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  postgres:15

Redis Cache:

docker run -d \
  -e REDIS_PASSWORD=redispass \
  -p 6379:6379 \
  redis:latest

Method 2: Using .env File (Professional Way)

For many variables, using -e flags gets messy. Use .env file instead.

Create .env file:
APP_ENV=production
APP_PORT=3000
APP_DEBUG=false
DATABASE_URL=postgresql://user:pass@db-host:5432/mydb
API_KEY=secret-key-abc123
JWT_SECRET=jwt-secret-xyz789
LOG_LEVEL=info
CACHE_ENABLED=true

Run with .env file:
docker run --env-file .env -d myapp:v1

All variables from .env file loaded into container.

Benefits:
- Cleaner command
- Reusable across runs
- Keep secrets out of shell history
- Easier management (all variables in one place)

Warning: Keep .env file secure (don't commit to git, use .gitignore).

.gitignore:
.env
.env.local
.env.*.local

Different .env files for different environments:

.env.dev (development):
APP_ENV=development
DEBUG=true
DATABASE_URL=localhost:5432

.env.prod (production):
APP_ENV=production
DEBUG=false
DATABASE_URL=prod-db.company.com:5432

Run:
docker run --env-file .env.dev myapp (dev)
docker run --env-file .env.prod myapp (prod)

Setting Default Values in Dockerfile:

Use ENV instruction to set defaults. Can override with -e at runtime.

Dockerfile:
FROM node:18
ENV PORT=3000
ENV NODE_ENV=production
COPY app.js /app/
WORKDIR /app
CMD ["node", "app.js"]

Running with defaults:
docker run -d myapp (uses PORT=3000, NODE_ENV=production)

Running with overrides:
docker run -d -e PORT=4000 -e NODE_ENV=development myapp (overrides defaults)

Dockerfile defaults provide fallback values. Runtime overrides take precedence.

Real-World Example: Multi-Tier Application

Database container:
docker run -d \
  --name postgres_db \
  -e POSTGRES_USER=appuser \
  -e POSTGRES_PASSWORD=dbpass123 \
  -e POSTGRES_DB=app_database \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:15

Backend API container:
docker run -d \
  --name backend \
  -e DATABASE_URL=postgresql://appuser:dbpass123@postgres_db:5432/app_database \
  -e PORT=5000 \
  -e DEBUG=false \
  -p 5000:5000 \
  backend-app:v1

Frontend container:
docker run -d \
  --name frontend \
  -e API_URL=http://backend:5000 \
  -e APP_ENV=production \
  -p 80:80 \
  frontend-app:v1

Each container gets its own environment configuration.

Environment Variables in CI/CD:

Secrets stored in CI/CD system (GitHub Actions, Jenkins, etc).

GitHub Actions example:
- name: Build and Push Image
  run: |
    docker build -t myapp:${{ github.sha }} .
    docker run -e DATABASE_PASSWORD=${{ secrets.DB_PASSWORD }} -e API_KEY=${{ secrets.API_KEY }} myapp:${{ github.sha }}

CI/CD injects secrets at runtime without hardcoding.

Summary:

Environment variables = dynamic configuration.

Benefits:
- Reuse same image for different environments
- Keep secrets safe (not in image)
- Change config without rebuild
- Flexibility at runtime
- Professional practice (required for production)

- Variables are scoped **only to the container**
- Host system variables are NOT changed
- Use **UPPER_CASE** naming convention for env variables (e.g., `MY_NAME`, `APP_ENV`)
- For secrets, prefer `--env-file` or Docker Secrets (advanced)

---

