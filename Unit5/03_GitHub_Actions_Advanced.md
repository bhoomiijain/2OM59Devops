# Unit V — GitHub Actions: Advanced Topics

## Actions (Reusable Components)

Actions are **pre-built, reusable units of code** you can use in your workflows.

```yaml
steps:
  - uses: actions/checkout@v3        # checks out your code
  - uses: actions/setup-java@v3      # installs Java
    with:
      java-version: '17'
  - uses: actions/upload-artifact@v4 # saves build output
```

---

## Runners

A **runner** is a server that executes the workflow.

| Type | Options |
|------|---------|
| GitHub-hosted | `ubuntu-latest`, `windows-latest`, `macos-latest` |
| Self-hosted | Your own server/machine |

```yaml
runs-on: ubuntu-latest   # GitHub provides and manages this
```

---

## Matrix Strategy (Parallel Testing)

Test across multiple configurations at once.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14, 16, 18]    # runs 3 jobs in parallel

    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
```

---

## Real-Life Example — Java CI Pipeline

```yaml
name: Java CI Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Java
        uses: actions/setup-java@v3
        with:
          java-version: '17'

      - name: Build with Maven
        run: mvn clean install
```

---

## Quiz Answers (from class slides)

| # | Question | Answer |
|---|----------|--------|
| 1 | Correct execution order | B — Workflow → Jobs → Steps |
| 2 | Reusable unit in GitHub Actions | C — Action |
| 3 | Workflow files are defined using | B — YAML file |
| 4 | Workflow files stored at | C — `.github/workflows/` |
| 5 | Component that defines WHEN workflow runs | C — Events |
| 6 | Valid push trigger syntax | B — `on: push: branches: [main]` |
| 7 | Role of a runner | B — Executes workflows |
| 8 | Jobs run by default | B — In parallel |
| 9 | Steps inside a job run | C — Sequentially |
| 10 | `cron: '0 0 * * *'` means | C — Every day at midnight |
| 11 | Test on multiple Node versions | B — Matrix strategy |
| 12 | Workflow fails at step 2, what happens | B — Job stops execution |
| 13 | GitHub-hosted runners support | C — Linux, Windows, macOS |

---

> 📅 *Unit V — CI with GitHub Actions (Part 2)*
> 📖 *Reference: Github_actions.pptx*
