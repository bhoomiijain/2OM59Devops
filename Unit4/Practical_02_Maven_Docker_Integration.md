# Practical 2 — Maven + Docker Integration

## Objective
Integrate Maven (Java build tool) with Docker so that Maven can **build the JAR and then build the Docker image** in one workflow.

---

## Tools Required
- Java JDK 17+
- Apache Maven
- Docker Desktop (running)

---

## Step 1 — Create Maven Project

```bash
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=docker-demo \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false
```

This generates the standard Maven folder structure:
```
docker-demo/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/example/
                └── App.java
```

---

## Step 2 — Modify the Java Program

Edit `src/main/java/com/example/App.java`:

```java
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Hello from Docker + Maven Integration!");
    }
}
```

---

## Step 3 — Create Dockerfile

In the **root of the project** (same level as `pom.xml`):

```dockerfile
FROM eclipse-temurin:21-jdk
WORKDIR /app
COPY target/docker-maven-demo-1.0-SNAPSHOT.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

> **Note:** The JAR filename must match what Maven generates — check `target/` after `mvn package`.

---

## Step 4 — Update pom.xml

Replace `pom.xml` contents with this (adds both the JAR plugin and Docker plugin):

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>docker-maven-demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>docker-maven-demo</name>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- JAR Plugin: sets main class -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.example.App</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>

            <!-- Docker Plugin: builds Docker image via Maven -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.4.13</version>
                <configuration>
                    <repository>docker-maven-demo</repository>
                    <tag>${project.version}</tag>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

---

## Step 5 — Build JAR File

```bash
mvn clean package
```

Output: `target/docker-maven-demo-1.0-SNAPSHOT.jar`

---

## Step 6 — Build Docker Image Using Maven

```bash
mvn dockerfile:build
```

This calls the Spotify Dockerfile Maven Plugin which:
1. Reads the `Dockerfile` in the project root
2. Builds the Docker image
3. Tags it as `docker-maven-demo:1.0-SNAPSHOT`

---

## Step 7 — Verify Docker Image

```bash
docker images
# Should show: docker-maven-demo   1.0-SNAPSHOT
```

---

## Step 8 — Run Docker Container

```bash
docker run docker-maven-demo:1.0-SNAPSHOT
# Output: Hello from Docker + Maven Integration!
```

---

## Troubleshooting — `mvn dockerfile:build` Fails

If you get a connection error, Docker's TCP socket isn't exposed. Fix:

1. Open **Docker Desktop**
2. Go to **Settings → General**
3. ✅ Enable: **"Expose daemon on tcp://localhost:2375 without TLS"**
4. Click **Apply & Restart**
5. Run `mvn dockerfile:build` again

---

## Summary — What Each Tool Does

| Tool | Role |
|------|------|
| Maven | Compiles Java, runs tests, packages JAR |
| Dockerfile | Defines how to containerize the JAR |
| `dockerfile-maven-plugin` | Bridges Maven and Docker — builds image as part of Maven lifecycle |
| Docker | Runs the final container |

---

> 📅 *Practical #2 — Unit IV (Maven + Docker Integration)*
> 📖 *Reference: UNIT_4_Practical_MVN_Docker_Integration.docx*
