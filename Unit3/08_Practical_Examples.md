# Topic 8: Practical Examples

**[← Back to Unit 3](README.md)**

## Overview

Learn to build real-world applications with Docker Compose through three complete examples.

---

## Example 1: Node.js + MongoDB Stack

### Scenario

Build a Node.js application that uses MongoDB for data storage with persistent storage and proper networking.

### Requirements

- Node.js application accessible on port 3000
- MongoDB accessible on port 27017
- MongoDB should start before Node.js app
- MongoDB data persists across container restarts
- App connects to MongoDB via environment variable
- Development workflow with live code reload

### Project Structure

```
project/
├── docker-compose.yml
├── Dockerfile
├── package.json
├── app.js
├── .dockerignore
└── src/
    ├── models/
    ├── routes/
    └── controllers/
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    container_name: node-app
    working_dir: /usr/src/app
    volumes:
      - .:/usr/src/app
      - /usr/src/app/node_modules
    command: npm start
    ports:
      - "3000:3000"
    environment:
      - MONGO_URL=mongodb://mongo:27017/mydb
      - NODE_ENV=development
      - PORT=3000
    depends_on:
      - mongo
    restart: unless-stopped
    networks:
      - app-network

  mongo:
    image: mongo:6
    container_name: mongo-db
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    restart: unless-stopped
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

volumes:
  mongo-data:

networks:
  app-network:
    driver: bridge
```

### Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm ci

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### package.json

```json
{
  "name": "nodejs-mongodb-app",
  "version": "1.0.0",
  "description": "Node.js + MongoDB application",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

### app.js

```javascript
const express = require('express');
const mongoose = require('mongoose');

const app = express();

// Connect to MongoDB
mongoose.connect(process.env.MONGO_URL, {
    useNewUrlParser: true,
    useUnifiedTopology: true
}).then(() => {
    console.log('✓ Connected to MongoDB');
}).catch(err => {
    console.error('✗ MongoDB connection error:', err);
    process.exit(1);
});

app.use(express.json());

// Health check endpoint
app.get('/health', (req, res) => {
    res.json({ status: 'ok', db: 'connected' });
});

// API routes
app.get('/api/test', (req, res) => {
    res.json({ message: 'Hello from Node.js' });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

### Usage

**Start Services:**
```bash
# Build and start
docker-compose up -d

# View status
docker-compose ps

# Stream logs
docker-compose logs -f app

# Access application
curl http://localhost:3000/api/test
# Response: { "message": "Hello from Node.js" }
```

**Development Workflow:**
```bash
# Make code changes on host machine
# Changes reflected in container (via volume mount)
# Nodemon restarts app automatically

# Enter container shell
docker-compose exec app bash
npm test
exit

# View MongoDB data
docker-compose exec mongo mongosh
> db.users.find()
> exit

# Stop services
docker-compose stop

# Resume services
docker-compose start

# Clean shutdown
docker-compose down
```

**Scaling:**
```bash
# Run 3 instances of Node.js app (behind load balancer)
docker-compose up --scale app=3 -d

# Verify
docker-compose ps
# Shows: app_1, app_2, app_3
```

---

## Example 2: WordPress + MySQL with Persistent Storage

### Scenario

Run a WordPress blog platform with MySQL database, persistent storage, and automatic backups.

### Requirements

- WordPress accessible on port 8080
- MySQL database with persistent storage
- Easy configuration via environment variables
- Auto-restart on failure
- Volume management for backups

### Project Structure

```
wordpress/
├── docker-compose.yml
├── .env
├── .env.example
├── backup.sh
└── data/
    └── (volumes mounted here)
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  wordpress:
    image: wordpress:6-apache
    container_name: wordpress-app
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: ${DB_USER}
      WORDPRESS_DB_PASSWORD: ${DB_PASSWORD}
      WORDPRESS_DB_NAME: ${DB_NAME}
      WORDPRESS_TABLE_PREFIX: wp_
    volumes:
      - wordpress-data:/var/www/html
      - ./wp-config-extra.php:/usr/local/etc/php/conf.d/uploads.ini
    depends_on:
      - db
    networks:
      - wordpress-network
    healthcheck:
      test: curl -f http://localhost/wp-admin/admin-ajax.php?action=heartbeat || exit 1
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: mysql:5.7
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    volumes:
      - db-data:/var/lib/mysql
      - ./db-init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"
    networks:
      - wordpress-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  wordpress-data:
  db-data:

networks:
  wordpress-network:
    driver: bridge
```

### .env File

```
DB_NAME=wordpress_db
DB_USER=wpuser
DB_PASSWORD=secure_password_123
DB_ROOT_PASSWORD=root_secure_password_456
```

### .env.example (Commit to Git)

```
DB_NAME=wordpress_db
DB_USER=wpuser
DB_PASSWORD=change_me
DB_ROOT_PASSWORD=change_me
```

### Usage

**Initial Setup:**
```bash
# Copy env template
cp .env.example .env

# Edit .env with actual passwords
nano .env

# Start services
docker-compose up -d

# Wait for WordPress to initialize
sleep 30

# Access WordPress
# Open browser: http://localhost:8080
# Complete WordPress setup wizard
```

**Accessing Services:**
```bash
# View logs
docker-compose logs -f wordpress

# Access database
docker-compose exec db mysql -u wpuser -p wordpress_db
# Enter password from .env

# Inside MySQL:
> SELECT * FROM wp_users;
> SELECT * FROM wp_posts;
> exit

# View WordPress files
docker-compose exec wordpress bash
ls -la /var/www/html
wp post list       # WordPress CLI
exit
```

**Backup:**
```bash
# Backup database
docker-compose exec db mysqldump -u wpuser -p wordpress_db > backup.sql
# (Enter password when prompted)

# Backup WordPress files
docker cp wordpress-app:/var/www/html ./wordpress-backup

# Both should be version-controlled/stored safely
```

**Restore:**
```bash
# Stop services
docker-compose down

# Remove volumes to start fresh
docker volume rm wordpress_wordpress-data
docker volume rm wordpress_db-data

# Start services
docker-compose up -d

# Restore database (wait for MySQL to be ready)
sleep 10
docker-compose exec -T db mysql -u wpuser -p wordpress_db < backup.sql
# (Enter password when prompted)

# Restore WordPress files
rm -rf wordpress-data/*
cp wordpress-backup/* wordpress-data/

# Verify
docker-compose ps
```

---

## Example 3: React + Spring Boot + PostgreSQL (3-Tier Application)

### Scenario

Complete full-stack application with React frontend, Spring Boot backend, and PostgreSQL database.

### Requirements

- React frontend on port 3000
- Spring Boot backend on port 8080
- PostgreSQL database on port 5432
- All services on same network
- Database initialization scripts
- Health checks before deployment

### Project Structure

```
full-stack-app/
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── package-lock.json
│   ├── public/
│   ├── src/
│   └── .dockerignore
├── backend/
│   ├── Dockerfile
│   ├── pom.xml
│   ├── src/
│   └── .dockerignore
└── database/
    └── init.sql
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  # React Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: react-app
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://backend:8080/api
      - NODE_ENV=production
    depends_on:
      - backend
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: curl -f http://localhost:3000 || exit 1
      interval: 30s
      timeout: 10s
      retries: 3

  # Spring Boot Backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: spring-boot-app
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/appdb
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=postgres123
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
      - SPRING_PROFILES_ACTIVE=docker
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: curl -f http://localhost:8080/actuator/health || exit 1
      interval: 30s
      timeout: 10s
      retries: 3

  # PostgreSQL Database
  db:
    image: postgres:15-alpine
    container_name: postgres-db
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres123
      POSTGRES_INITDB_ARGS: "-c max_connections=200"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network
    restart: unless-stopped

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

### Frontend Dockerfile

```dockerfile
# Build stage
FROM node:18-alpine as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM node:18-alpine
WORKDIR /app
RUN npm install -g serve
COPY --from=builder /app/build ./build
EXPOSE 3000
CMD ["serve", "-s", "build", "-l", "3000"]
```

### Backend Dockerfile

```dockerfile
# Build stage
FROM maven:3.8-openjdk-17 as builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY . .
RUN mvn clean package -DskipTests

# Runtime stage
FROM openjdk:17-jdk-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

### database/init.sql

```sql
-- Create schema and tables
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    description TEXT,
    stock INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(id),
    total_amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Insert sample data
INSERT INTO users (name, email) VALUES 
    ('John Doe', 'john@example.com'),
    ('Jane Smith', 'jane@example.com');

INSERT INTO products (name, price, description, stock) VALUES 
    ('Laptop', 999.99, 'High-performance laptop', 10),
    ('Mouse', 29.99, 'Wireless mouse', 50),
    ('Keyboard', 79.99, 'Mechanical keyboard', 30);
```

### Usage

**Build and Start:**
```bash
# Build images
docker-compose build

# Start all services
docker-compose up -d

# Verify status
docker-compose ps

# Expected output:
# NAME            STATUS      PORTS
# react-app       Up 2 mins   0.0.0.0:3000->3000/tcp
# spring-boot-app Up 1 min    0.0.0.0:8080->8080/tcp
# postgres-db     Up 3 mins   0.0.0.0:5432->5432/tcp
```

**Access Services:**
```bash
# Frontend
curl http://localhost:3000
# Or open browser: http://localhost:3000

# Backend
curl http://localhost:8080/api/products
# Returns: [{"id":1,"name":"Laptop",...}]

# Database
docker-compose exec db psql -U postgres -d appdb
> SELECT * FROM users;
> SELECT * FROM products;
> \q
```

**Monitoring:**
```bash
# View logs
docker-compose logs -f

# Frontend logs only
docker-compose logs -f frontend

# Backend logs only
docker-compose logs -f backend

# Database logs
docker-compose logs -f db
```

**Debugging:**
```bash
# Enter backend container
docker-compose exec backend bash
# Inside:
ls -la /app
java -version
cat app.jar

# Enter database
docker-compose exec db psql -U postgres -d appdb
> \dt                    # List tables
> SELECT * FROM orders;  # View orders
```

**Rebuild After Code Changes:**
```bash
# Frontend code changed
docker-compose build frontend
docker-compose up -d frontend

# Backend code changed
docker-compose build backend
docker-compose up -d backend
# (Database service doesn't rebuild)
```

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│              User's Browser                             │
└─────────────────────────────────────────────────────────┘
                    │
                    │ http://localhost:3000
                    ▼
┌─────────────────────────────────────────────────────────┐
│   React Frontend (Node.js)                              │
│   - User Interface                                      │
│   - API calls to http://backend:8080/api                │
│   - Port 3000 exposed to host                           │
└─────────────────────────────────────────────────────────┘
                    │
                    │ http://backend:8080/api
                    │ (Inside Docker network)
                    ▼
┌─────────────────────────────────────────────────────────┐
│   Spring Boot Backend (Java)                            │
│   - REST APIs                                           │
│   - Business Logic                                      │
│   - Database queries                                    │
│   - Port 8080 exposed to host                           │
└─────────────────────────────────────────────────────────┘
                    │
                    │ JDBC
                    │ jdbc:postgresql://db:5432/appdb
                    ▼
┌─────────────────────────────────────────────────────────┐
│   PostgreSQL Database                                   │
│   - User data, products, orders                         │
│   - Persistent storage                                  │
│   - Port 5432 exposed to host                           │
│   - Volume: postgres-data                               │
└─────────────────────────────────────────────────────────┘
```

### Communication Flow

```
1. User opens http://localhost:3000
   ↓
2. React app loads and makes API call
   → http://backend:8080/api/products
   
3. Inside Docker network, request reaches Spring Boot
   ↓
4. Spring Boot queries PostgreSQL
   → SELECT * FROM products
   
5. Database returns data
   ↓
6. Spring Boot processes and returns JSON
   → [{"id":1, "name":"Laptop", ...}]
   
7. React displays products in UI
   ↓
8. User sees product catalog
```

---

## Key Learnings from Examples

### Example 1 (Node + MongoDB)
- Volume mounting for live development
- Service dependency management
- Environment variable configuration
- Health checks for databases

### Example 2 (WordPress + MySQL)
- Data persistence across restarts
- Backup and restore procedures
- .env file management (secrets)
- Multi-tier networking

### Example 3 (React + Spring Boot + PostgreSQL)
- Multi-stage Docker builds
- Complex inter-service communication
- Database initialization with SQL scripts
- Full-stack application architecture

---

## Common Patterns

### Pattern 1: Database Initialization

```yaml
db:
  volumes:
    - ./init.sql:/docker-entrypoint-initdb.d/init.sql
# SQL file runs on first container start only
```

### Pattern 2: Health Checks

```yaml
healthcheck:
  test: ["CMD", "pg_isready", "-U", "postgres"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### Pattern 3: Environment Configuration

```yaml
environment:
  - VARIABLE=${VARIABLE:-default_value}
# Use .env file for secrets, defaults for non-secrets
```

### Pattern 4: Volume Management

```yaml
volumes:
  - named-volume:/container/path
  - ./host/path:/container/path
  - ./config:/container/config:ro
```

---

## Best Practices Summary

1. **Always tag images** with version
2. **Use health checks** in production
3. **Externalize configuration** via env files
4. **Version control** docker-compose.yml, not .env
5. **Use named volumes** for persistent data
6. **Keep containers minimal** (one process per container)
7. **Document** all services and their purpose
8. **Test locally** before production deployment

---

**[← Back to Unit 3](README.md)**
