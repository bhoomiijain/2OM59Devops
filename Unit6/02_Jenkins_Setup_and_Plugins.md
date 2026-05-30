# Unit VI — Jenkins: Setup, Installation & Plugins

## Jenkins Installation

### Installation Options

**Option A — Docker (recommended for practice)**
```bash
docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  --name jenkins \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

**Option B — JAR file**
```bash
java -jar jenkins.war
```

**Option C — System install**
Download installer from [jenkins.io](https://jenkins.io)

### First Access
1. Go to: `http://localhost:8080`
2. Enter the admin password (found at `/var/jenkins_home/secrets/initialAdminPassword`)
3. Install **Suggested Plugins**
4. Create your **admin user** (username, password, email)

---

## Creating Your First Job

1. Click **"New Item"**
2. Enter a name (e.g., `my-first-job`)
3. Choose **"Freestyle project"**
4. Configure:
   - **Source Code Management** → Git (enter repo URL)
   - **Build Trigger** → Poll SCM or on Push
   - **Build** → Execute Shell or Invoke Maven
5. Click **Save** → Click **Build Now**

> **Think of a Job as:** "Tell Jenkins what to do and when to do it."

---

## Plugins in Jenkins

**Plugins are add-ons** that extend Jenkins' core functionality. Without plugins, Jenkins is just a basic automation server.

### Important Plugins

| Plugin | Purpose |
|--------|---------|
| Git Plugin | Connect to GitHub/GitLab repos |
| Maven Integration | Build Java projects with Maven |
| Docker Plugin | Build and run Docker containers |
| Docker Pipeline | Use Docker in Pipeline scripts |
| Pipeline | Define CI/CD as code |
| HTML Publisher | Publish test reports |
| Role-based Authorization | User access control |
| SSH Agent | Deploy via SSH |
| Slack Notification | Send build alerts to Slack |

### How to Install a Plugin
1. Go to **Jenkins Dashboard**
2. Click **Manage Jenkins**
3. Click **Manage Plugins**
4. Search in the **Available** tab
5. Click **Install without restart** OR **Download now and install after restart**

> ⚠️ Too many plugins = performance issues or conflicts. Install only what you need.

---

## Required Plugins for Full CI/CD

These are essential for a complete CI/CD pipeline:

- Pipeline Graph Analysis
- Pipeline: REST API
- Pipeline: Stage View
- Maven Integration
- Docker Commons
- Docker Pipeline
- Git server
- SSH Agent
- HTML Publisher
- Role-based Authorization Strategy

---

> 📅 *Unit VI — CI/CD with Jenkins (Part 2)*
> 📖 *Reference: CICD_JENKINS_Unit_6.pptx*
