# Unit VI — Jenkins: Pipelines & CI/CD Workflows

## What is a Jenkins Pipeline?

A **Jenkins Pipeline** is a set of automated steps defined as code that Jenkins follows to build, test, and deploy your application.

> Think of it as a **scripted flowchart** for your DevOps process.

### Why Use Pipelines?
- Automate the **entire CI/CD process**
- Builds are **repeatable** and **version-controlled** (stored as code)
- Run complex workflows (parallel stages, conditions, etc.)
- Concept: **"Pipeline as Code"** — your build process lives in your repo

### Two Types of Pipelines

| Type | Description | Best For |
|------|-------------|----------|
| **Declarative** | Structured, easy to read/write | Beginners, standard workflows |
| **Scripted** | Flexible, uses Groovy | Advanced users, complex logic |

---

## Declarative Pipeline Syntax

```groovy
pipeline {
    agent any          // run on any available node

    tools {
        maven 'Maven'  // use Maven tool configured in Jenkins
    }

    stages {
        stage('Build') {
            steps {
                bat 'mvn clean package'   // Windows: bat | Linux: sh
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }
    }
}
```

---

## End-to-End CI/CD Pipeline: Jenkins + GitHub + Maven + Docker

### Project Structure

```
neww/
├── src/
│   └── main/java/com/example/
│       └── Application.java
├── pom.xml
├── Dockerfile
└── Jenkinsfile
```

### Application.java

```java
package com.example;

import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@SpringBootApplication
@RestController
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @GetMapping("/")
    public String home() {
        return "CI/CD Working!";
    }
}
```

### Dockerfile

```dockerfile
FROM openjdk:17
COPY target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

### Jenkinsfile

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Build') {
            steps {
                bat 'mvn clean package'
            }
        }
    }
}
```

### pom.xml (Spring Boot)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>app</artifactId>
    <version>1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## Push Project to GitHub

```bash
git init
git add .
git commit -m "added project files"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/neww.git
git push -u origin main
```

**Verify on GitHub — repo should contain:**
- `Jenkinsfile`
- `pom.xml`
- `Dockerfile`
- `src/` folder

---

## Configure Jenkins Pipeline from GitHub

1. **Jenkins Dashboard** → your project → **Configure**
2. Scroll to **Pipeline Section**
3. Set **Definition** → `Pipeline script from SCM`
4. Set **SCM** → `Git`
5. Enter **Repository URL**: `https://github.com/YOUR_USERNAME/neww.git`
6. Enter **Branch**: `*/main`
7. Enter **Script Path**: `Jenkinsfile`
8. Click **Save** → Click **Apply** → Click **Build Now**

---

## Jenkins vs GitHub Actions

| Feature | Jenkins | GitHub Actions |
|---------|---------|---------------|
| Hosting | Self-hosted | GitHub-hosted (cloud) |
| Setup | Manual install + config | Zero setup (built into GitHub) |
| Plugins | 1800+ plugins | Marketplace actions |
| Customization | Very high | High |
| Best for | Enterprise, complex pipelines | Open-source, GitHub projects |
| Cost | Free (self-hosted infra costs) | Free tier available |
| Pipeline format | Groovy (Jenkinsfile) | YAML (.github/workflows/) |

---

> 📅 *Unit VI — CI/CD with Jenkins (Part 3)*
> 📖 *Reference: CICD_JENKINS_Unit_6.pptx*
