# Practice Questions & Workflow Exercises — GitHub Actions

## Identify the Error (Q1)

**Buggy YAML:**
```yaml
name: CI Pipeline
on push:           # ❌ missing colon after 'on'
  branches:
    - main
jobs:
 build:
  runs-on ubuntu-latest    # ❌ missing colon after 'runs-on'
  steps:
    - name Checkout code   # ❌ missing colon after 'name'
      uses actions/checkout@v3   # ❌ missing colon after 'uses'
```

**Fixed YAML:**
```yaml
name: CI Pipeline
on:
  push:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
```

---

## Q2 — Trigger on `develop` Branch Only

```yaml
name: CI on Develop

on:
  push:
    branches:
      - develop

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: echo "Building on develop branch"
```

---

## Q3 — Minimal Workflow: Checkout + Hello World

```yaml
name: Hello World Workflow

on: push

jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Print Hello World
        run: echo "Hello World"
```

---

## Q4 — Full CI Workflow (Checkout → Build → Test)

```yaml
name: Full CI Workflow

on:
  push:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Build project
        run: echo "Building..."

      - name: Test project
        run: echo "Testing..."
```

---

## Q5 — Job with Install Dependencies + Run Tests

```yaml
name: Node CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

---

## Q6 — Identify What This Does

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build
        run: ls
```

**Answer:** Runs `ls` (lists files in current directory) on a fresh `ubuntu-latest` runner whenever triggered. Since there's no `actions/checkout` step, the directory will be nearly empty (just runner defaults).

---

## Practice Scenarios

### Scenario 1 — Set env variable + run container interactively
```bash
docker run -it --name my_app \
  -e APP_ENV=production \
  -v C:\app\data:/data \
  ubuntu /bin/bash

# Inside:
echo $APP_ENV                          # production
echo "Hello this is my first program" > /data/test.txt
cat /data/test.txt
echo "Second line added" >> /data/test.txt
cat /data/test.txt
exit
```

### Scenario 2 — Deploy MongoDB with port mapping
```bash
docker pull mongo
docker run -d --name DB-app -p 80:8082 mongo
```

### Scenario 3 — Nginx with env variable (interactive)
```bash
docker run -it \
  -e NGINX_PORT=8080 \
  -p 8080:80 \
  nginx bash
```

---

## GitHub Actions Workflow Cheatsheet

```yaml
# Full reference template
name: Workflow Name

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:

env:
  MY_VAR: value       # workflow-level env variable

jobs:
  job-name:
    runs-on: ubuntu-latest
    needs: other-job   # run after another job

    strategy:
      matrix:
        version: [16, 18, 20]   # parallel execution

    steps:
      - uses: actions/checkout@v4
      - name: My Step
        run: echo "Hello ${{ env.MY_VAR }}"
        env:
          STEP_VAR: step-level-value
```

---

> 📅 *Unit V — Practice & Exercises*
> 📖 *Reference: Github_actions.pptx slides 34–40*
