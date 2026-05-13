# Topic 6: Maven Plugins

**[← Back to Unit 4](README.md)**

## ⚙️ Understanding Plugins

Plugins extend Maven's functionality. Each build phase is actually a plugin execution.

> **Plugin:** A reusable Maven component that performs specific build tasks (compiling, testing, packaging, etc.).

---

## How Plugins Work

### Plugin Execution Model

```
mvn compile
    ↓
Maven looks in pom.xml for plugins bound to "compile" phase
    ↓
Executes plugins in order
    ↓
Each plugin performs its goals
    ↓
Output written to target/
```

### Plugin Binding

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <!-- Bound to compile phase by default -->
            <configuration>
                <source>11</source>
                <target>11</target>
            </configuration>
        </plugin>
    </plugins>
</build>

When you run "mvn compile":
  → maven-compiler-plugin executes
  → Compiles src/main/java to target/classes
```

---

## 🔨 Popular Maven Plugins

### 1. Maven Compiler Plugin

**Purpose:** Compile Java source code

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <source>11</source>
        <target>11</target>
        <encoding>UTF-8</encoding>
        <verbose>false</verbose>
        <!-- Fail on warnings (optional) -->
        <failOnWarning>false</failOnWarning>
        <!-- Compiler arguments -->
        <compilerArgs>
            <arg>-Xlint</arg>  <!-- Show warnings -->
        </compilerArgs>
    </configuration>
</plugin>
```

**Use Cases:**
- Set Java version (source/target)
- Configure encoding
- Add compiler flags

**Command:**
```bash
mvn compile              # Runs compiler plugin
```

---

### 2. Maven Surefire Plugin

**Purpose:** Execute unit tests

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <!-- Test classes to include -->
        <includes>
            <include>**/*Test.java</include>
            <include>**/*Tests.java</include>
            <include>**/Test*.java</include>
        </includes>
        
        <!-- Tests to exclude -->
        <excludes>
            <exclude>**/*IntegrationTest.java</exclude>
        </excludes>
        
        <!-- Test parallel execution -->
        <parallel>methods</parallel>
        <threadCount>4</threadCount>
        
        <!-- Skip tests -->
        <!-- <skipTests>true</skipTests> -->
    </configuration>
</plugin>
```

**Test Reports:**
```
target/surefire-reports/
├── TEST-com.example.ServiceTest.xml
└── com.example.ServiceTest.txt
```

**Command:**
```bash
mvn test                           # Run all tests
mvn test -Dtest=ServiceTest        # Run specific test
mvn test -DskipTests               # Skip tests
```

---

### 3. Maven Failsafe Plugin

**Purpose:** Execute integration tests

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <!-- Integration test classes -->
        <includes>
            <include>**/*IT.java</include>
            <include>**/*IntegrationTest.java</include>
        </includes>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**Difference from Surefire:**
```
Surefire (mvn test):
  ✓ Fast unit tests
  ✓ Isolated, no dependencies
  ✓ Failure stops build

Failsafe (mvn verify):
  ✓ Slower integration tests
  ✓ Test full application
  ✓ Can have setup/teardown
  ✓ Failure doesn't stop build (reports at end)
```

---

### 4. Maven Shade Plugin

**Purpose:** Create "fat JAR" with all dependencies included

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.4</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <finalName>my-app-with-deps</finalName>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.example.Main</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

**Normal JAR vs Fat JAR:**
```
Normal JAR (my-app-1.0.0.jar):
├── com/example/Main.class
├── application.properties
└── (not include dependencies)
  
Size: ~50 KB
Requires: All dependencies in classpath at runtime

Fat JAR (my-app-1.0.0-with-deps.jar):
├── com/example/Main.class
├── application.properties
├── org/springframework/... (all Spring classes)
├── org/postgresql/...     (all PostgreSQL classes)
└── (includes all dependencies)
  
Size: ~20 MB
Requires: Nothing! Self-contained!
```

**Use Cases:**
- Executable JAR files
- Docker containers
- Lambda functions
- Command-line tools

---

### 5. Maven Assembly Plugin

**Purpose:** Create distributions (ZIP, TAR, etc.)

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**Output:**
```
target/
├── my-app-1.0.0.jar
├── my-app-1.0.0-jar-with-dependencies.jar
└── my-app-1.0.0-sources.jar
```

---

### 6. Maven JAR Plugin

**Purpose:** Control JAR packaging and manifest

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <archive>
            <manifest>
                <!-- Main entry point -->
                <mainClass>com.example.Main</mainClass>
                <!-- Class-Path for dependencies -->
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
            </manifest>
            <!-- Add custom manifest entries -->
            <manifestEntries>
                <Build-Timestamp>${timestamp}</Build-Timestamp>
                <Version>${project.version}</Version>
                <Developer>build-system</Developer>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>
```

**Created MANIFEST.MF:**
```
Manifest-Version: 1.0
Main-Class: com.example.Main
Class-Path: lib/spring-core-5.0.0.jar lib/log4j-2.14.1.jar
Build-Timestamp: 2023-01-01T12:00:00Z
Version: 1.0.0
Developer: build-system
```

---

### 7. Maven Javadoc Plugin

**Purpose:** Generate API documentation

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>3.3.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**Output:**
```
target/site/apidocs/        (HTML documentation)
target/my-app-1.0.0-javadoc.jar   (Javadoc JAR)
```

**Command:**
```bash
mvn javadoc:javadoc        # Generate documentation
mvn site                   # Generate site with docs
```

---

### 8. Spring Boot Maven Plugin

**Purpose:** Build executable Spring Boot applications

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>2.7.0</version>
    <configuration>
        <mainClass>com.example.Application</mainClass>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>  <!-- Create executable JAR -->
            </goals>
        </execution>
    </executions>
</plugin>
```

**Creates:**
```
my-app-1.0.0.jar          (executable!)
  ├── BOOT-INF/classes/   (application code)
  ├── BOOT-INF/lib/       (all dependencies)
  ├── org/springframework/boot/loader/ (Spring Boot launcher)
  └── META-INF/MANIFEST.MF (entry point)
```

**Executable:**
```bash
# Can run directly!
java -jar my-app-1.0.0.jar

# No need for classpath, all dependencies included
```

---

## 🔧 Plugin Configuration

### Execution Configuration

```xml
<plugin>
    <groupId>org.example</groupId>
    <artifactId>my-plugin</artifactId>
    <version>1.0.0</version>
    
    <!-- Global plugin configuration -->
    <configuration>
        <param1>value1</param1>
        <param2>value2</param2>
    </configuration>
    
    <!-- Specific execution -->
    <executions>
        <execution>
            <id>execution-1</id>
            <phase>compile</phase>
            <goals>
                <goal>specific-goal</goal>
            </goals>
            <!-- Execution-specific configuration -->
            <configuration>
                <param1>override-value</param1>
            </configuration>
        </execution>
        
        <execution>
            <id>execution-2</id>
            <phase>package</phase>
            <goals>
                <goal>another-goal</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

---

## 🎯 Plugin Execution Examples

### Compile Phase

```
mvn compile
  ↓
Maven finds all plugins bound to "compile" phase:
  1. resources plugin (copy resources)
  2. compiler plugin (compile source)
  ↓
Executes in order
```

### Package Phase

```
mvn package
  ↓
Maven executes all phases up to package:
  1. validate → (no plugins)
  2. compile → compiler plugin
  3. test → surefire plugin
  4. package → jar plugin, shade plugin
  ↓
Creates target/my-app-1.0.0.jar
```

---

## 📋 Plugin Discovery

### Search for Plugins

```bash
# Search on Maven Central
mvn help:describe -Dplugin=org.apache.maven.plugins:maven-compiler-plugin

# Browse plugins
https://maven.apache.org/plugins/
```

### Common Plugin Groups

| Provider | Plugins |
|---|---|
| Apache Maven | compiler, jar, surefire, assembly, shade |
| Spring Boot | spring-boot-maven-plugin |
| Docker | docker-maven-plugin, jib-maven-plugin |
| Google | jib-maven-plugin |

---

## 🔑 Key Takeaway

> **Plugins are how Maven accomplishes build tasks. Understand core plugins and know how to configure them for your project's needs.**

**Remember:**
1. ✅ Plugins bind to lifecycle phases
2. ✅ Core plugins: compiler, surefire, jar, shade
3. ✅ Configure via `<configuration>` element
4. ✅ Create fat JARs with shade or Spring Boot plugin
5. ✅ Use failsafe for integration tests

---

## Next Topic

**[Read about Docker Integration with Maven →](07_Docker_Integration_with_Maven.md)**

Automate Docker image building with Maven plugins.
