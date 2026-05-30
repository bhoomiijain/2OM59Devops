# Unit V — GitHub Actions: Core Concepts

## What is GitHub Actions?

**GitHub Actions** is a built-in CI/CD platform inside GitHub that lets you **automate workflows directly in your repository**.

It helps you:
- Build code automatically on every push
- Run tests before merging
- Deploy applications
- Automate repetitive dev tasks

> **Analogy:** GitHub Actions is like a smart assistant that automatically performs tasks (testing, building, deploying) whenever something happens in your repository.

---

## Core Concepts

### What is a Workflow?
- An **automated process** defined in a `.yml` (YAML) file
- Stored at: `.github/workflows/` inside your repo
- Contains one or more **jobs**
- Each job has **steps**

### Workflow Directory Structure
```
your-repo/
├── .github/
│   └── workflows/
│       ├── build.yml
│       └── deploy.yml
├── src/
├── tests/
└── README.md
```

---

## Components of a Workflow

| Component | Role | Analogy |
|-----------|------|---------|
| **Event (on)** | When to run | The alarm trigger |
| **Job** | What to do | A task group |
| **Step** | How to do it | Individual instructions |
| **Action** | Reusable pre-built task | A ready-made tool |
| **Runner** | Server that executes it | The machine that does the work |

---

## Basic Workflow Structure

```yaml
name: My First Workflow

on: push          # Event: runs on every push

jobs:
  build:          # Job name
    runs-on: ubuntu-latest    # Runner type

    steps:
      - name: Checkout code
        uses: actions/checkout@v3    # Action (reusable)

      - name: Run a script
        run: echo "Hello World"      # Shell command
```

---

## Events (Triggers)

Events are activities that **trigger a workflow**.

| Event | When it fires |
|-------|--------------|
| `push` | Code is pushed to a branch |
| `pull_request` | PR is opened, updated, or merged |
| `workflow_dispatch` | Manually triggered from GitHub UI |
| `schedule` | Runs on a cron schedule |
| `issues` | When an issue is created/updated |

---

## Types of Triggers

### 1. Push Trigger — most common for CI
```yaml
on:
  push:
    branches: [main]
```

### 2. Pull Request Trigger
```yaml
on:
  pull_request:
    branches: [main]
```

### 3. Schedule Trigger (Cron)
```yaml
on:
  schedule:
    - cron: '0 0 * * *'   # Every day at midnight
```

### 4. Manual Trigger
```yaml
on:
  workflow_dispatch:
```

### 5. Branch + Path Filter
```yaml
on:
  push:
    branches: [main, dev]
    paths:
      - 'src/**'   # Only triggers if files in src/ changed
```

### 6. Tag-based Trigger
```yaml
on:
  push:
    tags:
      - 'v1.*'
```

---

## Jobs

A **job** is a set of steps that run on the same runner.

```yaml
jobs:
  build:              # job 1
    runs-on: ubuntu-latest
  test:               # job 2
    runs-on: ubuntu-latest
```

> By default, **jobs run in PARALLEL**. To run sequentially, use `needs`.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
  test:
    needs: build      # test runs AFTER build finishes
    runs-on: ubuntu-latest
```

---

## Steps

Steps inside a job run **sequentially** (one after another).

```yaml
steps:
  - name: Step 1
    run: echo "Hello"

  - name: Step 2
    run: ls -la
```

> If a step fails → the **job stops** at that point.

---

## Execution Order

```
Workflow
  └── Job 1 (runs-on: ubuntu-latest)
        └── Step 1 → Step 2 → Step 3 (sequential)
  └── Job 2 (runs-on: ubuntu-latest)    ← parallel with Job 1 by default
        └── Step 1 → Step 2
```

**Order summary:** `Workflow → Jobs (parallel) → Steps (sequential)`

---

> 📅 *Unit V — CI with GitHub Actions (Part 1)*
> 📖 *Reference: Github_actions.pptx*
