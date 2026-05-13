# Topic 8: Dockerizing Java Applications

**[← Back to Unit 4](README.md)**

## 🐳 Production-Ready Docker Images for Java

This final topic covers best practices for creating optimal Docker images for Java applications.

---

## ✅ Best Practices Checklist

```
Security:
  ☐ Use specific base image versions (not "latest")
  ☐ Run as non-root user
  ☐ Keep base image up-to-date
  ☐ Scan images for vulnerabilities
  ☐ Use minimal base images

Performance:
  ☐ Multi-stage builds
  ☐ Layer caching strategy
  ☐ Minimize image size
  ☐ Optimize JVM for containers
  ☐ Health checks

Maintainability:
  ☐ Clear Dockerfile
  ☐ Document exposed ports and volumes
  ☐ Version control all build files
  ☐ Consistent naming conventions
  ☐ Keep images small
```

---

## 🖼️ Base Image Selection

### Image Size Comparison

| Base Image | Size | Includes | Use Case |
|---|---|---|---|
| `openjdk:11` | ~650MB | Full JDK | Development |
| `openjdk:11-jre` | ~350MB | Only JRE | Most applications |
| `openjdk:11-jre-slim` | ~200MB | Minimal JRE | Production |
| `openjdk:11-jre-alpine` | ~80MB | Alpine Linux | Smallest size |
| `gcr.io/distroless/java11` | ~40MB | Only app + JRE | Minimal, secure |

### Choosing a Base Image

```dockerfile
# Development / Testing
# Full JDK for debugging
FROM openjdk:11

# Production Standard
# Minimal JRE, good security/size balance
FROM openjdk:11-jre-slim

# Production Optimized
# Smallest possible
FROM gcr.io/distroless/java11

# Custom optimization
# Build your own minimal base
FROM alpine:latest
RUN apk add --no-cache openjdk11-jre
```

---

## 📋 Bad vs Good Dockerfiles

### ❌ Bad Dockerfile (Problems)

```dockerfile
FROM openjdk:11  # Too large, full JDK

WORKDIR /app
COPY . .         # Copy EVERYTHING (unnecessary files)

RUN mvn clean package  # Build inside container (slow!)
                       # No layer caching

CMD ["java", "-jar", "target/app.jar"]  # No JVM optimization
```

**Problems:**
```
✗ Large base image (650MB)
✗ Rebuilds entire build on any code change
✗ All source files in image (bloat)
✗ Tests run in production image
✗ No JVM optimization
✗ Non-optimal JVM flags for containers
✗ Slow rebuilds (30+ seconds)
```

---

### ✅ Good Dockerfile (Multi-Stage)

```dockerfile
# Stage 1: Build
FROM maven:3.8.1-openjdk-11 AS builder

WORKDIR /build
COPY pom.xml .
# Download dependencies (cached)
RUN mvn dependency:go-offline

COPY src ./src
# Build application
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM openjdk:11-jre-slim

WORKDIR /app

# Create non-root user
RUN groupadd -r app && useradd -r -g app app

# Copy only JAR from builder
COPY --from=builder /build/target/*.jar /app/app.jar

# Set ownership
RUN chown -R app:app /app

# Switch to non-root user
USER app

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD java -cp /app/app.jar -c "... health check logic ..."

# JVM optimization for containers
ENV JAVA_OPTS="-Xmx512m -XX:+UseG1GC -XX:+UseStringDeduplication"

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
```

**Benefits:**
```
✓ Minimal base image (200MB)
✓ Only application code in production image
✓ No build tools in production
✓ Dependency layer cached separately
✓ Code changes only rebuild needed layers
✓ Non-root user (security)
✓ JVM optimized for containers
✓ Health check configured
✓ Fast rebuilds (~3 seconds)
```

---

## 🔧 JVM Optimization for Containers

### Container-Aware JVM Flags

```dockerfile
# Old way (before Java 10)
FROM openjdk:11-jre-slim
# Had to manually set memory limits
ENV JAVA_OPTS="-Xmx512m -Xms512m"
CMD java $JAVA_OPTS -jar app.jar

# New way (Java 10+)
FROM openjdk:11-jre-slim
# JVM automatically detects container limits!
ENV JAVA_OPTS="-XX:+UseContainerSupport \
               -XX:+UseG1GC \
               -XX:MaxRAMPercentage=75 \
               -XX:+UseStringDeduplication \
               -XX:+UnlockExperimentalVMOptions \
               -XX:G1NewCollectionHeapPercent=20"
CMD java $JAVA_OPTS -jar app.jar
```

### JVM Flag Explanations

| Flag | Purpose | Value |
|---|---|---|
| `-XX:+UseContainerSupport` | Detect Docker limits | Boolean (default: true in Java 10+) |
| `-XX:+UseG1GC` | G1 garbage collector | Good for large heaps |
| `-XX:MaxRAMPercentage=75` | Max heap % of RAM | 75% = good default |
| `-XX:+UseStringDeduplication` | String optimization | Saves memory |
| `-Xmx512m` | Max heap (old style) | Use MaxRAMPercentage instead |
| `-Xms256m` | Initial heap | Usually same as -Xmx |

### Memory Configuration

```dockerfile
# Container with 1GB RAM
# Kubernetes/Docker limit: 1024MB

# Option 1: Fixed allocation
ENV JAVA_OPTS="-Xmx768m"  # Use 75% of 1GB

# Option 2: Automatic (better)
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75"  # Automatically 75% of available

# Option 3: Container-native (best)
# Java 10+ automatically detects cgroup limits
# No configuration needed!
```

---

## 🏗️ Layer Optimization

### Layer Caching Strategy

```dockerfile
# Order matters for layer caching!

FROM maven:3.8.1-openjdk-11 AS builder

WORKDIR /build

# Layer 1: Copy only pom.xml (rarely changes)
COPY pom.xml .

# Layer 2: Download dependencies (cached unless pom.xml changes)
RUN mvn dependency:go-offline

# Layer 3: Copy source code (changes frequently)
COPY src ./src

# Layer 4: Build (only rebuilds if source or pom.xml changed)
RUN mvn clean package -DskipTests
```

**Cache Effectiveness:**
```
First build:
  Layer 1 (pom.xml):       0 sec (new)
  Layer 2 (dependencies):  45 sec (download)
  Layer 3 (source):        0 sec (new)
  Layer 4 (build):         30 sec (compile)
  Total: 75 seconds

Change one Java file:
  Layer 1 (pom.xml):       0 sec (cached!)
  Layer 2 (dependencies):  0 sec (cached!)
  Layer 3 (source):        0 sec (new)
  Layer 4 (build):         3 sec (incremental)
  Total: 3 seconds  ← 25x faster!
```

---

## 🔒 Security Best Practices

### Non-Root User

```dockerfile
FROM openjdk:11-jre-slim

# Create user and group
RUN groupadd -r app && useradd -r -g app app

# Copy application
COPY --from=builder /build/target/*.jar /app/app.jar

# Set ownership
RUN chown -R app:app /app

# Switch to user (before CMD)
USER app

CMD ["java", "-jar", "/app/app.jar"]
```

**Why Non-Root?**
```
Root User (Security Risk):
  ✗ Container breakout = full host access
  ✗ Attacker can install tools
  ✗ Modify host files
  ✗ Not recommended for production

Non-Root User (Secure):
  ✓ Container breakout = limited access
  ✓ Attacker confined to app user permissions
  ✓ Standard practice in production
```

### Vulnerability Scanning

```bash
# Scan image for vulnerabilities
docker scan my-app:1.0.0

# Example output:
# Vulnerabilities: 3
# ├── HIGH: CVE-2021-1234 in base image
# ├── MEDIUM: CVE-2021-5678 in library
# └── LOW: CVE-2021-9999 in dependency

# Update base image to fix
FROM openjdk:11-jre-slim:latest  # Update to latest patch
```

---

## ❤️ Health Checks

### Dockerfile Health Check

```dockerfile
FROM openjdk:11-jre-slim

COPY --from=builder /build/target/*.jar /app/app.jar

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

CMD ["java", "-jar", "/app/app.jar"]
```

**Health Check Phases:**
```
--interval=30s    Every 30 seconds check health
--timeout=3s      Wait max 3 seconds for response
--start-period=40s Give app 40 seconds to start before checking
--retries=3       Mark unhealthy after 3 failures
```

### Spring Boot Health Endpoint

```java
// Spring Boot provides /actuator/health by default
// GET /actuator/health returns:
// {"status":"UP"}

// Dockerfile uses this
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:8080/actuator/health || exit 1
```

---

## 📊 Image Size Optimization

### Before and After

```
Bad Dockerfile Image:
  openjdk:11                      650 MB
  + Maven cache                  ~200 MB
  + Source code                  ~100 MB
  + Build artifacts              ~150 MB
  = Total                       ~1100 MB

Good Multi-Stage Image:
  openjdk:11-jre-slim            200 MB
  + Application JAR              ~100 MB
  = Total                        ~300 MB

Reduction: 3.7x smaller!
```

### Size Reduction Techniques

```dockerfile
# Before: 500MB
FROM openjdk:11
RUN apt-get update && apt-get install -y curl wget git
COPY . .

# After: 210MB (multi-stage)
FROM maven:3.8.1-openjdk-11 AS builder
COPY . .
RUN mvn clean package

FROM openjdk:11-jre-slim
COPY --from=builder /build/target/*.jar /app/app.jar

# After: 120MB (distroless)
FROM maven:3.8.1-openjdk-11 AS builder
COPY . .
RUN mvn clean package

FROM gcr.io/distroless/java11
COPY --from=builder /build/target/*.jar /app/app.jar
```

---

## 🔧 Complete Production Dockerfile

```dockerfile
# Stage 1: Build
FROM maven:3.8.1-openjdk-11 AS builder

LABEL stage=builder

WORKDIR /build

# Copy pom.xml only (maximize caching)
COPY pom.xml .

# Download dependencies (cached layer)
RUN mvn dependency:go-offline -B

# Copy source code
COPY src ./src

# Build application
RUN mvn clean package -DskipTests -q

# Stage 2: Runtime
FROM openjdk:11-jre-slim

LABEL maintainer="devops@example.com"
LABEL description="My Java Application"
LABEL version="1.0.0"

WORKDIR /app

# Create non-root user
RUN groupadd -r app && useradd -r -g app app

# Copy only JAR from builder
COPY --from=builder --chown=app:app /build/target/*.jar /app/app.jar

# Copy optional configuration
COPY --chown=app:app config/ /app/config/

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8080/actuator/health || exit 1

# Container environment
ENV JAVA_OPTS="-XX:+UseContainerSupport \
               -XX:MaxRAMPercentage=75 \
               -XX:+UseG1GC \
               -XX:+UseStringDeduplication"

# Switch to non-root user
USER app

# Expose port
EXPOSE 8080

# Entry point
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
```

---

## 🚀 Deployment Checklist

```
Before Shipping to Production:

Build:
  ☐ Multi-stage Dockerfile
  ☐ Layer caching optimized
  ☐ Image scanned for vulnerabilities
  ☐ Size under 300MB

Security:
  ☐ Runs as non-root user
  ☐ No build tools in image
  ☐ Base image up-to-date
  ☐ No secrets in image

Runtime:
  ☐ JVM optimized for containers
  ☐ Health checks configured
  ☐ Environment variables for config
  ☐ Logging configured
  ☐ Resource limits set (CPU, memory)

Kubernetes/Orchestration (if applicable):
  ☐ Readiness probe configured
  ☐ Liveness probe configured
  ☐ Resource requests/limits set
  ☐ Graceful shutdown handling
```

---

## 🔑 Key Takeaway

> **Production Docker images for Java require careful attention to security, performance, and best practices. Multi-stage builds, non-root users, and JVM optimization create optimal containers.**

**Essential Practices:**
1. ✅ Use multi-stage builds (separate builder and runtime)
2. ✅ Choose minimal base images (jre-slim or distroless)
3. ✅ Run as non-root user
4. ✅ Optimize JVM for containers
5. ✅ Implement health checks
6. ✅ Scan for vulnerabilities
7. ✅ Keep images small
8. ✅ Cache layers effectively

**Result:** Fast, secure, optimal Docker images ready for production deployment.

---

## Unit 4 Complete! 🎉

You've learned:
- ✅ Why build tools exist (Topic 1)
- ✅ How to configure projects (Topic 2)
- ✅ Maven directory structure (Topic 3)
- ✅ Build lifecycle phases (Topic 4)
- ✅ Dependency management (Topic 5)
- ✅ Maven plugins (Topic 6)
- ✅ Docker integration (Topic 7)
- ✅ Production Docker images (Topic 8)

**Next Steps:**
- Practice building Java applications with Maven
- Dockerize a Spring Boot application
- Set up CI/CD pipelines
- Deploy to Kubernetes or Docker Swarm

Good luck! 🚀
