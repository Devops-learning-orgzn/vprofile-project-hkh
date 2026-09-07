
# Automated End-to-End Java CI/CD Pipeline

A production-simulated, enterprise-grade CI/CD pipeline built to automate the integration, security scanning, artifact management, and cloud deployment of a Java web application. This architecture bridges continuous development with automated infrastructure operations.

### Architecture diagram

```text
           [ Developer ]
                 │
                 ▼ (Pushes Code)
         ┌───────────────┐
         │    GitHub     │ ──────────┐ (Webhook)
         └───────────────┘           │
                                     ▼
         ┌──────────────────────────────────────────────┐
         │                 Jenkins CI                   │
         └──────┬──────────────┬──────────────┬─────────┘
                │              │              │
                ▼              ▼              ▼
         ┌───────────┐   ┌───────────┐   ┌───────────┐
         │SonarQube  │   │   Nexus   │   │  Docker   │
         │(Quality   │   │(Artifact  │   │(Container │
         │ Gates)    │   │  Storage) │   │  Build)   │
         └───────────┘   └───────────┘   └─────┬─────┘
                                               │
                                               ▼
                                         ┌───────────┐
                                         │   Trivy   │
                                         │(Security  │
                                         │  Scan)    │
                                         └─────┬─────┘
                                               │
                                               ▼ (Validated Image)
                                         ┌───────────┐
                                         │  AWS ECR  │
                                         └─────┬─────┘
                                               │
                                               ▼ (Update via jq)
         ┌───────────┐                   ┌───────────┐
         │   Slack   │ ◄──────────────── │  AWS ECS  │
         │ (Alerts)  │  (Status Update)  │ (Deploy)  │
         └───────────┘                   └───────────┘
```




### The 8-Step Execution Loop
1. **Source Control:** Developer pushes code to GitHub, triggering an automated webhook.
2. **Build Automation:** Jenkins polls the repository and compiles the Java application using Maven.
3. **Static Code Analysis:** Checkstyle and SonarQube evaluate code formatting, code quality gates, and bugs.
4. **Artifact Storage:** Compiled build packages (`vprofile-v2.war`) are version-controlled and uploaded to Nexus Repository Manager.
5. **Containerization:** A multi-stage Docker build packages the runtime environment efficiently.
6. **Security Auditing:** Trivy scans the container image layers for known CVE vulnerabilities before push.
7. **Cloud Deployment:** Pushes validated images to Amazon ECR and updates Amazon ECS task definitions/services via the AWS CLI.
8. **Feedback Loop:** Real-time build, test, and deployment statuses are forwarded to a dedicated Slack channel.

### Tech Stack & Lab Infrastructure
* **CI/CD Orchestration:** Jenkins
* **Cloud Infrastructure:** AWS (ECS, ECR, EC2, IAM)
* **Containerization:** Docker
* **Code Quality & Testing:** SonarQube, Maven, Checkstyle
* **Artifact Management:** Nexus Repository Manager
* **Security & Compliance:** Trivy
* **Alerts:** Slack Notifications

---

## 🛠️ 2. Infrastructure Setup (3 Linux Servers)

To eliminate performance cross-interference and simulate a real-world enterprise environment, the infrastructure is split across three distinct Linux instances.

| Server | Purpose | Core Requirements & Software Installed |
| :--- | :--- | :--- |
| **Server 1** | **Jenkins Build Machine** | Java (JDK 17 or 21), Maven 3.9.9, Docker Engine, Trivy Scanner, AWS CLI, `jq` |
| **Server 2** | **SonarQube Server** | Automated deployment via user data script (`userdata/sonar-setup.sh`) |
| **Server 3** | **Nexus Repository** | Automated deployment via user data script (`userdata/nexus-setup.sh`) |

*Note: Access to administrative UI consoles is managed via the public IP addresses of their respective instances using specific security group ports.*

---

## ⚙️ 3. Jenkins Server Configuration

Once your Jenkins Linux server is running, the following plugins and credentials must be configured to establish connectivity with external APIs and services.

### 🔹 Required Plugins
Navigate to **Manage Jenkins** ➡️ **Plugins** ➡️ **Available Plugins** and install:
* **Git** & **GitHub Integration** (To fetch source code repositories)
* **Maven Integration** & **Pipeline Maven Integration** (To compile Java applications)
* **SonarQube Scanner** (To analyze code quality and enforce quality gates)
* **Nexus Artifact Uploader** (To publish builds to Nexus)
* **Docker Pipeline** (To build, tag, and push container images)
* **Slack Notification** (To send real-time operational alerts)
* **Pipeline: AWS Steps** (Required for the `withAWS` block to handle secure ECS deployments)

### 🔹 Required Credentials
Navigate to **Manage Jenkins** ➡️ **Credentials** ➡️ **System** ➡️ **Global credentials** and register these exact IDs:
* `gitcreds` ➡️ Username and Password/Token for GitHub
* `sonacreds` ➡️ Secret text token generated from your SonarQube profile
* `nexuscreds` ➡️ Admin username and password for Nexus Repository Manager
* `slackcreds` ➡️ Integration secret token generated from Slack
* `awscreds` ➡️ AWS Access Key ID and Secret Access Key with appropriate IAM policies for ECR/ECS

---

## ☁️ 4. AWS Cloud Infrastructure (Manual Provisioning)

Before executing the pipeline, log into your AWS Console and manually configure the following target deployment platform resources:

1. **Amazon ECR:** Create a private registry repository to securely host your application's compiled Docker images.
2. **Amazon ECS Setup:**
   * **ECS Cluster:** Create a logical grouping for your containerized tasks.
   * **ECS Task Definition:** Blueprint specifying the ECR image path, environment variables, CPU, and Memory limits.
   * **ECS Service:** Configured to run and auto-scale instances of your task definition safely inside the cluster using a rolling update strategy.

---

## 📋 5. Pipeline Stages Walkthrough

The automation workflow processes the application across the following sequential phases:

* **Build & Unit Test:** Compiles the application via Maven using custom `settings.xml` configurations and runs standard test suites.
* **Code Inspections:** Runs Checkstyle validation and sends deep analysis metrics directly to the `sonarserver`.
* **Quality Gate Check:** Blocks the pipeline for up to 1 hour if the code coverage or quality metrics fail the predefined SonarQube safety limits.
* **Artifact Upload:** Packs the build into a standard Java Web Archive (`vprofile-v2.war`) and uploads it to the `vprofile-release` repository inside Nexus.
* **Container Security & Build:** Builds a container image via a multi-stage Dockerfile and parses it using `Trivy` to block deployments containing `HIGH` or `CRITICAL` vulnerabilities.
* **ECR Registry Upload:** Signs in securely using your `awscreds` profile and pushes image tags (`latest` and the dynamic `$BUILD_NUMBER`) to Amazon ECR.
* **ECS Rolling Deployment:** Dynamically extracts the active task configuration schema, modifies the image parameter target pointing to the newest ECR payload using `jq`, registers a fresh revision, triggers an update to the ECS service, and monitors cluster health until deployment is successful.

---

## 💡 6. Key Implementation Highlights

### 🐳 Multi-Stage Docker Optimization
Leveraged multi-stage Docker builds to dramatically reduce final deployment footprints. By separating the Maven build dependency workspace from the lightweight runtime container, the final image size was stripped down to the bare essentials. This design choice minimizes the infrastructure resource overhead and heavily reduces the image's security attack surface area.

### 🛡️ Continuous Security & Quality Gates
Enforced automated guardrails inside the execution flow. The pipeline is architected to immediately fail the build stage if SonarQube quality gate thresholds are breached or if Trivy detects critical vulnerability exposures in system layers. This ensures zero bad code or insecure containers reach production.

### 💬 Real-Time Slack Feedback Loop
Automated build status updates are dispatched at the conclusion of every execution loop:
* **Target Slack Channel:** `#devopscicd`
* **Alert Format:** Dispatches color-coded statuses (**Green** for `SUCCESS`, **Red** for `FAILURE`) embedded with active links pointing straight back to your Jenkins run console.
