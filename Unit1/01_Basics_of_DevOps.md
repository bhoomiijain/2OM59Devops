# DevOps Basics & Why Docker?

DevOps = Dev + Ops working together, not separately. Get code to production faster with collaboration and automation of build, test, deploy.

**Why DevOps?**

Traditional: developers write code, operations team deploys. Slow, lots of communication delays, blame games when things break.

DevOps: single team owns entire lifecycle. Automate everything. Deploy multiple times per day. Fix issues fast.

**DevOps vs Agile/Lean:**

Agile = development methodology (sprint-based, iterative development). Focuses on writing code fast.

Lean = eliminate waste, do more with less resources. Focuses on efficiency.

DevOps = combines both PLUS adds operations. Not just write code fast, but deploy fast too. Automate infrastructure. Monitor production. Quick feedback loops.

**DevOps tooling for each stage:**

1. Plan: Jira, Azure Boards, GitHub Projects
2. Code: GitHub, GitLab, Bitbucket (version control)
3. Build: Maven, Gradle, npm (compile & package code)
4. Test: JUnit, pytest, Selenium (automated testing)
5. Deploy: Docker, Kubernetes, Jenkins, Ansible (containerize & deploy)
6. Monitor: Prometheus, ELK Stack, Grafana (watch production)
7. Feedback: logs, metrics, alerts (continuous improvement)

**This course focuses on:**

Docker (build) + GitHub Actions (CI) + Jenkins (CD)

**Before Docker:**

monolithic apps on servers that get really heavy with VMs. Slow startup, "works on my machine" problem kills everything when you try to go live.

**Containerization:**

containers + microservices. Same image runs everywhere. Seconds to start. Less resources. Actually portable.

**Solves:**
- "works on my machine" problem (same container everywhere)
- Slow deployment (seconds vs minutes)
- Resource waste (one container vs full VM)
- Scalability (hundreds of containers vs few VMs)

**Course INT332 units:**
- Containers basics
- Images, Docker, Dockerfile
- Docker Compose
- Maven
- GitHub Actions (CI)
- Jenkins (CI/CD)

**Marks breakdown:**

Project 30, Git repo 30, Minor 5, AT2 45

**Need Docker Desktop or AWS to run stuff**

**Key terms:**
- Container = lightweight isolated environment for apps
- Image = blueprint/template for containers
- Docker = popular containerization tool
- DevOps = culture + tools for fast delivery
- CI/CD = continuous integration/continuous deployment
- Microservices = small independent services instead of one big app


