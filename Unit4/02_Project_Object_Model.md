# Topic 2: Project Object Model (POM)

**[← Back to Unit 4](README.md)**

## Overview

The POM (pom.xml) is the core configuration file for every Maven project. It's a declarative description of your project and how to build it.

---

## What is pom.xml?

**POM (Project Object Model):**
> The heart of every Maven project. A single XML file that describes your project, its dependencies, plugins, and build configuration.

**Why Called "Object Model":**
- Maven parses pom.xml into a Java object
- Plugins interact with this object
- Unified data model across all builds

**Key Principle: Declarative, Not Imperative**

```xml
<!-- Declarative: What you want to build -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.0</version>
    </dependency>
</dependencies>

<!-- vs Imperative: How to build it (old way) -->
<!-- download spring-core-5.0.0.jar from repo
     extract dependencies
     add to classpath
     compile...
     etc -->
```

---

## 📋 Core POM Elements

| Element | Purpose | Example | Required |
|---|---|---|---|
| **modelVersion** | POM format version | 4.0.0 | ✓ Yes |
| **groupId** | Organization identifier | com.example | ✓ Yes |
| **artifactId** | Project name | my-app | ✓ Yes |
| **version** | Release version | 1.0.0-SNAPSHOT | ✓ Yes |
| **packaging** | Output format | jar, war, ear, pom | Default: jar |
| **name** | Human-readable name | My Application | Optional |
| **description** | Project description | Web application | Optional |
| **dependencies** | Required libraries | `<dependency>...</dependency>` | Optional |
| **build** | Build configuration | plugins, source dirs | Optional |
| **properties** | Variables | `<maven.compiler.source>11</maven.compiler.source>` | Optional |
| **profiles** | Environment configs | dev, prod | Optional |

---

## Basic pom.xml Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Project Identification -->
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    
    <name>My Application</name>
    <description>A sample Maven application</description>
    
    <!-- Properties -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    
    <!-- Dependencies -->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <!-- Build Configuration -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    
</project>
```

---

## 🏷️ Project Identification

### groupId

**Purpose:** Unique identifier for your organization/company

```xml
<groupId>com.example</groupId>
```

**Convention:** Reverse domain name
- `com.example` → example.com company
- `org.apache` → Apache Foundation
- `com.github.username` → GitHub user
- `io.spring` → Spring.io

**Why Reverse?** Prevents naming conflicts across internet

**Examples:**
```
com.google              → Google
org.springframework     → Spring Framework
com.twitter            → Twitter
org.apache.commons     → Apache Commons
```

### artifactId

**Purpose:** Project/module name within organization

```xml
<artifactId>my-app</artifactId>
```

**Convention:** Lowercase, hyphen-separated

**Combined with groupId = Unique Project Identifier**
```
Full Identifier: com.example:my-app
Repository Path: com/example/my-app/
```

### version

**Purpose:** Release version of your artifact

```xml
<version>1.0.0-SNAPSHOT</version>
```

**Version Formats:**

| Format | Type | Meaning |
|---|---|---|
| `1.0.0` | Release | Stable production release |
| `1.0.0-SNAPSHOT` | Development | Work-in-progress, changes |
| `1.0.0-BETA` | Pre-release | Testing phase |
| `1.0.0-RC1` | Release Candidate | Almost ready |

**Semantic Versioning (MAJOR.MINOR.PATCH):**
```
1.0.0
│ │ │
│ │ └─ PATCH (bug fixes, backward compatible)
│ └─── MINOR (new features, backward compatible)
└───── MAJOR (breaking changes)

1.0.0 → 1.0.1  (patch: bug fix)
1.0.0 → 1.1.0  (minor: new feature)
1.0.0 → 2.0.0  (major: breaking change)
```

### packaging

**Purpose:** Output artifact type

```xml
<packaging>jar</packaging>
```

**Common Types:**

| Type | Output | Use Case |
|---|---|---|
| `jar` | .jar file | Java library or application |
| `war` | .war file | Web application |
| `ear` | .ear file | Enterprise application |
| `pom` | No artifact | Multi-module parent |
| `maven-plugin` | Plugin JAR | Maven extension |

---

## 📦 Dependencies Section

**Purpose:** Declare external libraries your project needs

### Simple Dependency

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.0</version>
    </dependency>
</dependencies>
```

### Multiple Dependencies

```xml
<dependencies>
    <!-- Web Framework -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.7.0</version>
    </dependency>
    
    <!-- Database -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.3.1</version>
    </dependency>
    
    <!-- Testing -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
    </dependency>
    
    <!-- Logging -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.11</version>
    </dependency>
</dependencies>
```

### Dependency Scopes

```xml
<!-- Compile: Include in all classpaths (default) -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.0.0</version>
    <scope>compile</scope>  <!-- or omit (default) -->
</dependency>

<!-- Test: Only for testing -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>

<!-- Runtime: Needed at runtime, not compile -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
    <scope>runtime</scope>
</dependency>

<!-- Provided: Environment provides it (e.g., servlet-api) -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
    <scope>provided</scope>
</dependency>
```

---

## ⚙️ Build Section

**Purpose:** Configure how Maven builds your project

### Basic Build Configuration

```xml
<build>
    <plugins>
        <!-- Compiler Plugin -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>11</source>
                <target>11</target>
            </configuration>
        </plugin>
        
        <!-- Surefire Plugin (Testing) -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.2</version>
            <configuration>
                <includes>
                    <include>**/*Test.java</include>
                    <include>**/*Tests.java</include>
                </includes>
            </configuration>
        </plugin>
        
        <!-- JAR Plugin -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.2.0</version>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>com.example.Main</mainClass>
                    </manifest>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Source Directories

```xml
<build>
    <sourceDirectory>src/main/java</sourceDirectory>
    <testSourceDirectory>src/test/java</testSourceDirectory>
    
    <resources>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
    </resources>
    
    <testResources>
        <testResource>
            <directory>src/test/resources</directory>
        </testResource>
    </testResources>
    
    <outputDirectory>target/classes</outputDirectory>
    <testOutputDirectory>target/test-classes</testOutputDirectory>
</build>
```

---

## 🔧 Properties Section

**Purpose:** Define variables to reuse throughout pom.xml

### Common Properties

```xml
<properties>
    <!-- Encoding -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    
    <!-- Java Version -->
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    
    <!-- Dependency Versions -->
    <spring.version>5.0.0</spring.version>
    <junit.version>4.13.2</junit.version>
    <log4j.version>2.14.1</log4j.version>
</properties>
```

### Using Properties

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>  <!-- Reference property -->
    </dependency>
    
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
    </dependency>
</dependencies>
```

**Benefits:**
- ✅ Single place to update versions
- ✅ Consistent dependency versions
- ✅ Easy to maintain

---

## 👨‍👩‍👧 Inheritance and Parent POM

**Parent POM Pattern:** Multi-module projects share configuration

```xml
<!-- parent-pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <properties>
        <spring.version>5.0.0</spring.version>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${spring.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>

<!-- child-project/pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Reference parent -->
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0.0</version>
    </parent>
    
    <artifactId>child-project</artifactId>
    
    <!-- Inherits parent properties and dependencies -->
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <!-- Version inherited from parent! -->
        </dependency>
    </dependencies>
</project>
```

---

## 🎯 Complete Real-World Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.mycompany</groupId>
    <artifactId>order-service</artifactId>
    <version>2.1.0</version>
    <packaging>jar</packaging>
    
    <name>Order Service</name>
    <description>REST API for order management</description>
    
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <spring-boot.version>2.7.0</spring-boot.version>
        <junit.version>4.13.2</junit.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring-boot.version}</version>
        </dependency>
        
        <!-- Spring Boot Data JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>${spring-boot.version}</version>
        </dependency>
        
        <!-- PostgreSQL Driver -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.3.1</version>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Logging -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.11</version>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>${spring-boot.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- Compiler Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
            
            <!-- Spring Boot Maven Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
            </plugin>
        </plugins>
    </build>
    
</project>
```

---

## 🔑 Key Takeaway

> **The pom.xml file is your project's single source of truth for build configuration, dependencies, and how to compile your application.**

**Key Points:**
1. ✅ Declarative configuration (what, not how)
2. ✅ Identifies project uniquely (groupId:artifactId:version)
3. ✅ Manages all dependencies automatically
4. ✅ Configures build plugins and parameters
5. ✅ Reusable across team and CI/CD

---

## Next Topic

**[Read about Maven Directory Structure →](03_Maven_Directory_Structure.md)**

Understand the standard layout Maven expects for Java projects.
