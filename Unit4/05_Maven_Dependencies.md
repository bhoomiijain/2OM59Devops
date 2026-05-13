# Topic 5: Maven Dependencies

**[← Back to Unit 4](README.md)**

## 📦 Dependency Management

A dependency is an external library your project needs. Maven automatically downloads and manages them.

---

## Declaring Dependencies

### Basic Dependency Declaration

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.0</version>
        <scope>compile</scope>        <!-- Optional: default is compile -->
    </dependency>
</dependencies>
```

### Multiple Dependencies

```xml
<dependencies>
    <!-- Spring Framework -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.0</version>
    </dependency>
    
    <!-- PostgreSQL Driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.3.1</version>
    </dependency>
    
    <!-- JUnit Testing -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 🎯 Dependency Scopes

Scopes determine when and how a dependency is used. This is critical for build correctness and JAR size.

### Scope Comparison Table

| Scope | Compile | Runtime | Packaged in JAR | Purpose |
|---|---|---|---|---|
| **compile** | ✓ | ✓ | ✓ | Application code uses it |
| **runtime** | ✗ | ✓ | ✓ | Only needed at runtime (JDBC drivers) |
| **provided** | ✓ | ✗ | ✗ | Environment provides it (Servlet API) |
| **test** | ✓ | ✗ | ✗ | Only for tests (JUnit) |
| **system** | ✓ | ✓ | Rarely | On specific system path |
| **import** | N/A | N/A | N/A | Import dependency management (POM files) |

---

## Scope Details

### 1. Compile Scope (Default)

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.0.0</version>
    <!-- scope omitted = compile (default) -->
</dependency>
```

**When Used:**
- Application code directly uses the library
- Must be available for compilation
- Must be in final JAR

**Example Use Cases:**
- Web frameworks (Spring, Spring Boot)
- ORM frameworks (Hibernate)
- HTTP clients
- JSON libraries (Jackson)
- Logging (SLF4J)

**Classpaths:**
```
Compile time:  ✓ Included
Test time:     ✓ Included
Runtime:       ✓ Included
JAR/WAR:       ✓ Included
```

---

### 2. Runtime Scope

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.3.1</version>
    <scope>runtime</scope>
</dependency>
```

**When Used:**
- Code doesn't directly reference the library
- Only needed when application runs
- Not needed for compilation

**Example Use Cases:**
- JDBC drivers (PostgreSQL, MySQL, Oracle)
- Runtime implementations of interfaces
- Plugin libraries

**Real Example:**
```java
// Code doesn't directly reference PostgreSQL driver
Connection conn = DriverManager.getConnection(
    "jdbc:postgresql://localhost/mydb",
    "user",
    "password"
);

// PostgreSQL driver loaded automatically at runtime
// No compile-time reference needed
```

**Classpaths:**
```
Compile time:  ✗ NOT Included
Test time:     ✓ Included
Runtime:       ✓ Included
JAR/WAR:       ✓ Included
```

---

### 3. Provided Scope

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
    <scope>provided</scope>
</dependency>
```

**When Used:**
- Application server provides the library
- Available for compilation but not packaged
- Prevents duplicate libraries in deployment

**Example Use Cases:**
- Servlet API (provided by Tomcat, Jetty)
- Java EE APIs (provided by app server)
- J2EE libraries

**Real Example - WAR File Deployment:**
```
Your WAR file:
  ├── WEB-INF/classes/ (your code)
  ├── WEB-INF/lib/     (your dependencies, but NOT servlet-api)
  └── (other resources)

Tomcat Server:
  lib/
    ├── servlet-api-2.5.jar    ← Provided by server!
    └── (other server libs)

Result: No duplicate libraries, no version conflicts
```

**Classpaths:**
```
Compile time:  ✓ Included
Test time:     ✓ Included
Runtime:       ✗ NOT Included (app server provides)
JAR/WAR:       ✗ NOT Included (reduces size)
```

---

### 4. Test Scope

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

**When Used:**
- Only used in test code
- Not available in application code
- Prevents test libraries in production

**Example Use Cases:**
- JUnit (unit testing)
- Mockito (mocking)
- TestNG (testing framework)
- Spring Test

**Real Example:**
```java
// In src/test/java/...
import org.junit.Test;           // ✓ Available
import static org.junit.Assert.*;

@Test
public void testCalculation() {
    assertEquals(4, 2 + 2);
}

// In src/main/java/... NOT available!
// import org.junit.Test;  ← Compilation ERROR
```

**Classpaths:**
```
Compile time:  ✗ NOT Included
Test time:     ✓ Included
Runtime:       ✗ NOT Included
JAR/WAR:       ✗ NOT Included (stays out of production)
```

---

## 🔗 Transitive Dependencies

When you depend on a library, Maven automatically includes its dependencies.

### Transitive Dependency Tree

```
Your Project
    └── Spring Core 5.0.0
        ├── commons-logging 1.2
        ├── commons-lang 3.0
        └── spring-jcl 5.0.0

Your Project depends on:
✓ spring-core-5.0.0.jar
✓ commons-logging-1.2.jar      ← Transitive!
✓ commons-lang-3.0.jar         ← Transitive!
✓ spring-jcl-5.0.0.jar         ← Transitive!
```

### Viewing Dependency Tree

```bash
# See all dependencies (including transitive)
mvn dependency:tree

# Output:
# [INFO] my-app:my-app:jar:1.0.0
# [INFO] +- org.springframework:spring-core:jar:5.0.0:compile
# [INFO] |  +- commons-logging:commons-logging:jar:1.2:compile
# [INFO] |  +- commons-lang:commons-lang:jar:3.0:compile
# [INFO] |  \- org.springframework:spring-jcl:jar:5.0.0:compile
# [INFO] +- junit:junit:jar:4.13.2:test
```

### Excluding Transitive Dependencies

```xml
<!-- Sometimes transitive dependencies cause conflicts -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.0.0</version>
    <exclusions>
        <!-- Don't include commons-logging -->
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- Instead use another logging library -->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.11</version>
</dependency>
```

---

## 📌 Dependency Management

### Using dependencyManagement

For multi-module projects, centralize version management:

```xml
<!-- parent-pom.xml -->
<project>
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <!-- Centralized version management -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>5.0.0</version>
            </dependency>
            
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.13.2</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>

<!-- child-pom.xml -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent</artifactId>
        <version>1.0.0</version>
    </parent>
    
    <artifactId>child</artifactId>
    
    <!-- No need to specify version! -->
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <!-- Version inherited from parent -->
        </dependency>
        
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <!-- Version inherited from parent -->
        </dependency>
    </dependencies>
</project>
```

**Benefits:**
```
✓ Single place to update versions
✓ All modules use same versions
✓ Prevents version conflicts
✓ Easy to upgrade library versions
```

---

## 🏷️ Version Management

### Version Formats

| Format | Type | Example |
|---|---|---|
| **Release** | Stable | 1.0.0, 2.1.5 |
| **SNAPSHOT** | Development | 1.0.0-SNAPSHOT |
| **Beta/Alpha** | Pre-release | 1.0.0-BETA, 1.0.0-RC1 |

### SNAPSHOT vs Release

**SNAPSHOT Version:**
```xml
<version>1.0.0-SNAPSHOT</version>

Characteristics:
✓ Work-in-progress
✓ Can change anytime
✓ Deployed to snapshots repository
✓ Maven checks for updates frequently
✓ Used during development
```

**Release Version:**
```xml
<version>1.0.0</version>

Characteristics:
✓ Stable, production-ready
✓ Never changes after release
✓ Deployed to releases repository
✓ Safe for production use
✓ Tagged in version control
```

### Version Ranges

```xml
<!-- Specific version -->
<version>1.0.0</version>

<!-- Hard lower bound -->
<version>[1.0.0,)</version>          <!-- >= 1.0.0 -->

<!-- Hard upper bound -->
<version>(,2.0.0)</version>           <!-- < 2.0.0 -->

<!-- Range -->
<version>[1.0.0,2.0.0)</version>      <!-- 1.0.0 <= x < 2.0.0 -->

<!-- Exclusive range -->
<version>(1.0.0,2.0.0)</version>      <!-- 1.0.0 < x < 2.0.0 -->
```

---

## 🔍 Finding Dependencies

### Maven Central Repository

```
https://mvnrepository.com/
https://search.maven.org/

Search by name or artifact:
- Group: org.springframework
- Artifact: spring-core
- Version: 5.0.0 (or latest)
```

### Dependency Snippet

Copy-paste into pom.xml:
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.0.0</version>
</dependency>
```

---

## Dependency Best Practices

```
1. ✓ Use specific versions (not ranges)
2. ✓ Scope dependencies correctly
3. ✓ Exclude unnecessary transitive deps
4. ✓ Use dependencyManagement in multi-module
5. ✓ Keep dependencies up-to-date
6. ✓ Review large dependency trees
7. ✗ Don't use version ranges for releases
8. ✗ Don't mix SNAPSHOT and release versions
```

---

## 🔑 Key Takeaway

> **Maven's dependency management automates what was once manual jar management. Use scopes correctly to ensure clean builds and deployments.**

**Key Points:**
1. ✅ Declare dependencies in pom.xml
2. ✅ Maven downloads transitive dependencies
3. ✅ Use correct scopes (compile, runtime, provided, test)
4. ✅ Manage versions centrally (dependencyManagement)
5. ✅ Exclude conflicting transitive dependencies when needed

---

## Next Topic

**[Read about Maven Plugins →](06_Maven_Plugins.md)**

Understand how plugins extend Maven's functionality.
