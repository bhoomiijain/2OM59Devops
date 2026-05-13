# Topic 4: Maven Build Lifecycle

**[← Back to Unit 4](README.md)**

## 🔄 Understanding Build Lifecycle

Maven's build lifecycle is a sequence of well-defined phases that execute in order. Each phase represents a stage in the build process.

---

## Build Lifecycle Phases

Maven defines three standard lifecycles. The most common is the **Default Lifecycle**:

```
default lifecycle:
  validate → compile → test → package → verify → install → deploy
  
clean lifecycle:
  pre-clean → clean → post-clean
  
site lifecycle:
  pre-site → site → post-site → site-deploy
```

---

## Default Lifecycle Phases

| Phase | Number | Purpose | Input | Output |
|---|---|---|---|---|
| **validate** | 1 | Check if project is correct | pom.xml | ✓/✗ valid |
| **compile** | 2 | Compile source code | src/main/java | target/classes/*.class |
| **test** | 3 | Run unit tests | src/test/java | test-reports, test-classes |
| **package** | 4 | Create distribution | compiled classes | .jar, .war, .ear |
| **verify** | 5 | Run integration tests | packaged artifact | ✓/✗ passes |
| **install** | 6 | Install to local repo | artifact | ~/.m2/repository/ |
| **deploy** | 7 | Upload to remote repo | artifact | remote repository |

---

## 🚀 Executing Phases

### Running Single Phase

```bash
# Phase execution
mvn compile              # Runs: validate, compile
mvn test                 # Runs: validate, compile, test
mvn package              # Runs: validate, compile, test, package
mvn install              # Runs: all default phases up to install
mvn deploy               # Runs: all default phases up to deploy
```

**Important:** Each phase runs previous phases

```
mvn package
  ↓
Automatically runs:
  1. validate
  2. compile
  3. test
  4. package
  (verify and install NOT run)
```

### Common Command Combinations

```bash
# Development: compile and test
mvn clean compile test

# Building: clean, compile, test, package
mvn clean package

# Installing locally for other projects
mvn clean install

# Full deployment pipeline
mvn clean deploy

# Generate documentation
mvn clean site
```

---

## 🔍 Detailed Phase Descriptions

### Phase 1: Validate

**Purpose:** Validate project structure and configuration

```bash
mvn validate
```

**Checks:**
- ✓ pom.xml is well-formed XML
- ✓ Required elements present (groupId, artifactId, version)
- ✓ Project structure correct
- ✓ Dependencies available

**Example Output:**
```
[INFO] BUILD SUCCESS
```

---

### Phase 2: Compile

**Purpose:** Compile Java source code to bytecode

```bash
mvn compile
```

**What Happens:**
```
src/main/java/
  └── com/example/Main.java
  
        ↓ javac
        
target/classes/
  └── com/example/Main.class
```

**Example Output:**
```
[INFO] Compiling 15 source files to /target/classes
[INFO] BUILD SUCCESS
```

**Classpath Resolution:**
```
Maven automatically:
✓ Downloads dependencies
✓ Adds to classpath
✓ Compiles with correct classpath
✓ No manual -cp flags needed!
```

---

### Phase 3: Test

**Purpose:** Compile and run unit tests

```bash
mvn test
```

**What Happens:**
```
1. Compile application
   src/main/java → target/classes

2. Compile tests
   src/test/java → target/test-classes

3. Run tests
   junit/testng on target/test-classes
   with classpath: target/classes:target/test-classes:dependencies

4. Generate reports
   target/surefire-reports/
```

**Example Output:**
```
[INFO] Compiling 5 test sources to /target/test-classes
[INFO] 
[INFO] -------T E S T S------
[INFO] Running com.example.ServiceTest
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] Total time: 2.345 s
[INFO] BUILD SUCCESS
```

**Skip Tests (when needed):**
```bash
mvn package -DskipTests
# or
mvn package -Dmaven.test.skip=true
```

---

### Phase 4: Package

**Purpose:** Create distributable artifact (JAR, WAR, etc.)

```bash
mvn package
```

**What Happens:**
```
target/classes/     (compiled code)
  +
target/test-classes/
  +
src/main/resources/ (config files)
  +
MANIFEST.MF
  
        ↓ jar/war/ear tool
        
target/my-app-1.0.0.jar
```

**JAR Contents:**
```
my-app-1.0.0.jar
├── META-INF/
│   ├── MANIFEST.MF
│   └── maven/com.example/my-app/pom.xml
├── com/
│   └── example/
│       ├── Main.class
│       ├── Service.class
│       └── Util.class
└── application.properties
```

**Example MANIFEST.MF:**
```
Manifest-Version: 1.0
Automatic-Module-Name: com.example.myapp
Created-By: Maven
Built-By: developer
Build-Jdk: 11.0.11
Main-Class: com.example.Main
Class-Path: lib/spring-core-5.0.0.jar lib/log4j-api-2.14.jar
```

---

### Phase 5: Verify

**Purpose:** Run integration tests on packaged artifact

```bash
mvn verify
```

**What Happens:**
```
1. Run all previous phases (validate, compile, test, package)

2. Run integration tests
   src/test/java/*IT.java
   Against packaged artifact

3. Generate integration test reports
   target/failsafe-reports/
```

**Integration Tests vs Unit Tests:**
```
Unit Tests (test phase):
  ✓ Fast (~ms each)
  ✓ Test single class in isolation
  ✓ Run during mvn test

Integration Tests (verify phase):
  ✓ Slower (seconds each)
  ✓ Test multiple components together
  ✓ May use database, network
  ✓ Run during mvn verify
  ✓ File pattern: *IT.java or *IntegrationTest.java
```

---

### Phase 6: Install

**Purpose:** Install artifact to local Maven repository

```bash
mvn install
```

**What Happens:**
```
target/my-app-1.0.0.jar
  ↓
Copy to ~/.m2/repository/

~/.m2/repository/com/example/my-app/1.0.0/
  ├── my-app-1.0.0.jar
  ├── my-app-1.0.0.pom
  ├── my-app-1.0.0-sources.jar
  └── my-app-1.0.0.jar.md5
```

**Why Local Repository?**
```
Other local projects can now depend on it:
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
</dependency>

Maven finds it in ~/.m2/repository/
```

**Example Output:**
```
[INFO] Installing /project/target/my-app-1.0.0.jar 
       to ~/.m2/repository/com/example/my-app/1.0.0/my-app-1.0.0.jar
[INFO] BUILD SUCCESS
```

---

### Phase 7: Deploy

**Purpose:** Upload artifact to remote Maven repository

```bash
mvn deploy
```

**What Happens:**
```
target/my-app-1.0.0.jar
  ↓
Upload to remote repository (Nexus, Artifactory, etc.)

Remote Repository:
  repo.example.com/maven/
    └── com/example/my-app/1.0.0/
        ├── my-app-1.0.0.jar
        ├── my-app-1.0.0.pom
        └── my-app-1.0.0-sources.jar
```

**Configuration in pom.xml:**
```xml
<distributionManagement>
    <repository>
        <id>internal-repo</id>
        <name>Internal Repository</name>
        <url>https://repo.company.com/nexus/repository/releases/</url>
    </repository>
    
    <snapshotRepository>
        <id>internal-snapshots</id>
        <name>Internal Snapshots</name>
        <url>https://repo.company.com/nexus/repository/snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

**Credentials in settings.xml:**
```xml
<!-- ~/.m2/settings.xml -->
<servers>
    <server>
        <id>internal-repo</id>
        <username>deployer</username>
        <password>secret-password</password>
    </server>
</servers>
```

---

## 🧹 Clean Lifecycle

Used to delete build outputs

```bash
mvn clean              # Remove target/ directory
mvn clean package      # Clean, then build
mvn clean install      # Clean, build, install
```

**Phases:**
1. **pre-clean** - Execute plugins before clean
2. **clean** - Delete target/ directory
3. **post-clean** - Execute plugins after clean

---

## 📊 Real Build Workflow

### Development Workflow

```bash
# Make code changes
vim src/main/java/com/example/Main.java

# Test locally
mvn clean test

# Fix bugs found in tests
# Repeat: make changes → test → fix

# Ready to package
mvn clean package

# Install for other projects to use
mvn clean install
```

### CI/CD Pipeline

```bash
# Pull code from Git
git clone repo

# Run full verification
mvn clean verify

# Deploy to staging
mvn deploy -P staging

# Run integration tests on staging
mvn -P staging integration-tests

# Deploy to production
mvn deploy -P production
```

### Multi-Module Project

```bash
# Project structure
parent-project/
├── pom.xml (packaging: pom)
├── module-a/
│   └── pom.xml
├── module-b/
│   └── pom.xml
└── module-c/
    └── pom.xml

# Build all modules
cd parent-project
mvn clean install

# Maven builds in dependency order:
# 1. Validates parent
# 2. Builds module-a
# 3. Builds module-b (depends on a)
# 4. Builds module-c (depends on a, b)
```

---

## 🎯 Goals vs Phases

**Phase:** A stage in the build lifecycle

```bash
mvn compile    # Phase: compile
mvn test       # Phase: test
mvn package    # Phase: package
```

**Goal:** A specific task/plugin execution

```bash
mvn compiler:compile           # Run compiler plugin's compile goal
mvn surefire:test             # Run surefire plugin's test goal
mvn jar:jar                   # Run jar plugin's jar goal
mvn help:describe             # Run help plugin's describe goal
```

**Format:** `mvn plugin-prefix:goal-name` or `mvn plugin-group:plugin-artifact:goal`

---

## 🔑 Key Takeaway

> **Maven's build lifecycle provides a standardized, predictable sequence of phases. Understand them to leverage Maven's power.**

**Key Phases to Remember:**
1. ✅ `mvn clean` - Clean build outputs
2. ✅ `mvn compile` - Compile source
3. ✅ `mvn test` - Run tests
4. ✅ `mvn package` - Create JAR/WAR
5. ✅ `mvn install` - Install locally
6. ✅ `mvn deploy` - Upload to repository

**Common Commands:**
- `mvn clean package` - Most common during development
- `mvn clean install` - When creating libraries
- `mvn clean verify` - In CI/CD pipelines
- `mvn clean deploy` - For releases

---

## Next Topic

**[Read about Maven Dependencies →](05_Maven_Dependencies.md)**

Understand scopes, transitive dependencies, and version management.
