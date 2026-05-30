# Practical 4 — Java CI Pipeline with Maven & GitHub Actions

## Problem Statement
Create a `.github/workflows/ci.yml` that:
- Checks out source code
- Sets up JDK 21
- Installs dependencies using Maven
- Compiles and runs unit tests
- Archives the generated JAR/WAR artifact

---

## Workflow File — `.github/workflows/ci.yml`

```yaml
name: Java CI with Maven

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'

      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-m2

      - name: Build with Maven
        run: mvn -B clean install

      - name: Run Tests
        run: mvn test

      - name: Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: java-build-artifact
          path: target/*.jar
```

---

## Step-by-Step Explanation

| Step | Action | Why |
|------|--------|-----|
| `actions/checkout@v4` | Clones the repo onto the runner | Runner starts empty — needs your code |
| `actions/setup-java@v4` | Installs JDK 21 (Temurin distribution) | Java needed to compile and run Maven |
| `actions/cache@v4` | Caches `~/.m2` Maven packages | Speeds up builds — avoids re-downloading dependencies |
| `mvn -B clean install` | Compiles, tests, packages JAR | `-B` = batch mode (no interactive prompts) |
| `mvn test` | Explicitly re-runs unit tests | Separate visible step for test results |
| `actions/upload-artifact@v4` | Saves `target/*.jar` as a downloadable artifact | Artifacts can be downloaded from the Actions run page |

---

## Cache Key Explained

```yaml
key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
```

- `runner.os` → Linux/Windows/macOS
- `hashFiles('**/pom.xml')` → fingerprint of all pom.xml files
- If `pom.xml` changes → cache is invalidated → full re-download
- If `pom.xml` unchanged → cache restored → fast build

---

## Triggers Explained

```yaml
on:
  push:
    branches: [ "main" ]      # Run CI on every push to main
  pull_request:
    branches: [ "main" ]      # Run CI on every PR targeting main
```

This ensures:
- Every direct commit to `main` is validated ✅
- Every PR going into `main` is validated before merge ✅

---

## Artifact Download

After the workflow runs:
1. Go to GitHub → Actions → your workflow run
2. Scroll to **Artifacts** section at bottom
3. Download `java-build-artifact` → contains the built `.jar`

---

> 📅 *Practical #4 — Unit V (Java CI with Maven + GitHub Actions)*
> 📖 *Reference: Practical_3.docx (numbered Practical #4 in file)*
