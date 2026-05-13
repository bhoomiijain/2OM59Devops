# Topic 7: Docker Integration with Maven

**[← Back to Unit 4](README.md)**

## 🐳 Automating Docker Builds with Maven

Instead of manually building Docker images, Maven plugins automate the process.

---

## Why Integrate Docker with Maven?

### Before: Manual Docker Build

```bash
# 1. Compile application manually
javac -cp lib/* src/main/java/com/example/Main.java -d target/classes

# 2. Create JAR manually
jar cvf target/my-app.jar -C target/classes .

# 3. Create Dockerfile manually
vim Dockerfile

# 4. Build Docker image manually
docker build -t my-app:1.0.0 .

# 5. Push to registry manually
docker tag my-app:1.0.0 registry.example.com/my-app:1.0.0
docker push registry.example.com/my-app:1.0.0

# Many manual steps, error-prone!
```

### After: Maven Automation

```bash
# Single command!
mvn clean package docker:build

# Maven automatically:
✓ Compiles code
✓ Creates JAR
✓ Builds Docker image
✓ Tags with version
✓ Ready for registry

# Single command to push
mvn docker:push
```

---

## 🔧 Docker Maven Plugin Options

### Option 1: fabric8 docker-maven-plugin

**Most Comprehensive:**

```xml
<plugin>
    <groupId>io.fabric8</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>0.40.0</version>
    <configuration>
        <images>
            <image>
                <name>${docker.registry}/my-app:${project.version}</name>
                <build>
                    <contextDir>${project.basedir}</contextDir>
                    <dockerFile>${project.basedir}/Dockerfile</dockerFile>
                </build>
                <run>
                    <!-- Runtime configuration -->
                </run>
            </image>
        </images>
    </configuration>
</plugin>
```

**Commands:**
```bash
mvn docker:build        # Build image
mvn docker:run          # Run container
mvn docker:push         # Push to registry
mvn docker:stop         # Stop running container
```

---

### Option 2: Google Jib Maven Plugin

**Best for Java Applications - No Dockerfile Needed!**

```xml
<plugin>
    <groupId>com.google.cloud.tools</groupId>
    <artifactId>jib-maven-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <!-- Base image -->
        <from>
            <image>openjdk:11-jre-slim</image>
        </from>
        
        <!-- Target image -->
        <to>
            <image>${docker.registry}/my-app:${project.version}</image>
            <!-- Credentials for registry -->
            <auth>
                <username>${docker.username}</username>
                <password>${docker.password}</password>
            </auth>
        </to>
        
        <!-- Container configuration -->
        <container>
            <mainClass>com.example.Main</mainClass>
            <jvmFlags>
                <jvmFlag>-Xmx512m</jvmFlag>
                <jvmFlag>-XX:+UseG1GC</jvmFlag>
            </jvmFlags>
            <ports>
                <port>8080</port>
            </ports>
            <environment>
                <JAVA_OPTS>-Xmx512m</JAVA_OPTS>
            </environment>
        </container>
    </configuration>
</plugin>
```

**Why Jib is Great:**
```
✓ No Dockerfile needed
✓ Automatically optimizes layers
✓ Builds efficiently
✓ Can push directly to registry
✓ Reproducible builds
✓ Layer caching for faster rebuilds
```

**Commands:**
```bash
mvn jib:build              # Build and save locally
mvn jib:dockerBuild        # Build to Docker daemon
mvn jib:build -Djib.to=repo/image:tag    # Push to registry
```

---

### Option 3: spotify docker-maven-plugin

**Lightweight and Simple:**

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.2.2</version>
    <configuration>
        <imageName>${docker.registry}/my-app:${project.version}</imageName>
        <dockerDirectory>${project.basedir}/src/main/docker</dockerDirectory>
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```

---

## 📋 Complete Jib Example

### Step 1: Add Jib Plugin to pom.xml

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-spring-boot-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.0</version>
    </parent>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- Spring Boot Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            
            <!-- Jib Docker Plugin -->
            <plugin>
                <groupId>com.google.cloud.tools</groupId>
                <artifactId>jib-maven-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <from>
                        <image>openjdk:11-jre-slim</image>
                    </from>
                    <to>
                        <image>registry.example.com/my-app:${project.version}</image>
                    </to>
                    <container>
                        <mainClass>com.example.Application</mainClass>
                        <ports>
                            <port>8080</port>
                        </ports>
                        <jvmFlags>
                            <jvmFlag>-Xmx512m</jvmFlag>
                        </jvmFlags>
                    </container>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### Step 2: Build Docker Image

```bash
# Build to local Docker daemon (requires Docker running)
mvn clean package jib:dockerBuild

# Or build and push directly to registry
mvn clean package jib:build
```

### Step 3: Run Container

```bash
# If built locally
docker run -p 8080:8080 registry.example.com/my-app:1.0.0

# Application starts!
```

---

## 🐳 Jib Generated Docker Image Structure

### What Jib Creates

```
Docker Image: registry.example.com/my-app:1.0.0
├── Base Layer: openjdk:11-jre-slim
│   (Java runtime, ~200MB)
│
├── Application Layer 1: Dependencies
│   /app/libs/spring-core-5.0.0.jar
│   /app/libs/log4j-2.14.1.jar
│   (cached, reused on rebuilds)
│
├── Application Layer 2: Resources
│   /app/resources/application.properties
│   /app/resources/logback.xml
│
├── Application Layer 3: Classes
│   /app/classes/com/example/Application.class
│   /app/classes/com/example/Service.class
│
└── Entry Point: java -cp ... com.example.Application
```

**Why Multiple Layers?**
```
✓ Dependencies rarely change → Cached
✓ Resources sometimes change → Separate layer
✓ Classes often change → Top layer
✓ Only changed layers rebuilt
✓ Much faster rebuilds!
```

---

## 🔄 Multi-Stage Build with Maven

For non-Java projects or complex builds:

```xml
<!-- Dockerfile -->
FROM maven:3.8.1-openjdk-11 AS builder

WORKDIR /build
COPY . .
RUN mvn clean package

# Production image
FROM openjdk:11-jre-slim
COPY --from=builder /build/target/*.jar /app/app.jar
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

**Why Multi-Stage?**
```
Builder Stage:
  - Includes Maven, compiler, test files
  - Large (~1GB+)
  - Only used for building

Production Stage:
  - Only includes JRE and compiled JAR
  - Small (~200MB)
  - Deployed to production
```

---

## 📋 Complete Docker Integration Workflow

### Scenario: Build and Push to Registry

```bash
# 1. Compile, test, create JAR
mvn clean package

# 2. Build Docker image
mvn jib:build -Djib.to.auth.username=myuser \
              -Djib.to.auth.password=mypass

# Result:
# ✓ Image built
# ✓ Pushed to registry.example.com/my-app:1.0.0
# ✓ Ready for deployment

# 3. Deploy from registry
docker run -p 8080:8080 registry.example.com/my-app:1.0.0
```

### CI/CD Pipeline Integration

```bash
#!/bin/bash
# CI/CD Script

# Checkout code
git clone repo

# Build and push to registry
mvn clean package \
    jib:build \
    -Djib.to.image=registry.company.com/my-app:$BUILD_VERSION \
    -Djib.to.auth.username=$REGISTRY_USER \
    -Djib.to.auth.password=$REGISTRY_PASS

# Automated Docker image deployed!
```

---

## 🔑 Key Takeaway

> **Maven Docker plugins automate Docker image building and pushing, integrating containerization into your build pipeline.**

**Best Choices:**
1. ✅ **Jib** - Best for Java apps, no Dockerfile needed
2. ✅ **fabric8** - Most comprehensive, full control
3. ✅ **spotify** - Simple, lightweight

**Benefits:**
- ✓ Automated Docker builds
- ✓ Version-controlled Docker images
- ✓ Part of CI/CD pipeline
- ✓ Reproducible builds
- ✓ Layer caching for speed

---

## Next Topic

**[Read about Dockerizing Java Applications →](08_Dockerizing_Java_Applications.md)**

Best practices for production-ready Docker images.
