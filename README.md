# Automated End-to-End Java CI/CD Pipeline

A production-simulated, enterprise-grade CI/CD pipeline built to automate the integration, security scanning, artifact management, and cloud deployment of a Java web application. This architecture bridges continuous development with automated infrastructure operations.

## Architecture Workflow
1. **Source Control:** Developer pushes code to GitHub triggering a webhook.
2. **Build Automation:** Jenkins polls the repository and compiles the Java app using Maven.
3. **Static Code Analysis:** Checkstyle & SonarQube evaluate code formatting, code quality gates, and bugs.
4. **Artifact Storage:** Compiled build packages (.war file) are version-controlled and uploaded to Nexus Repository Manager.
5. **Containerization:** Multi-stage Docker builds packages the runtime environment efficiently.
6. **Security Auditing:** Trivy scans the container image layers for known CVE vulnerabilities.
7. **Cloud Deployment:** Pushes validated images to Amazon ECR and updates Amazon ECS task definitions/services via AWS CLI.
8. **Feedback Loop:** Real-time build, test, and deployment statuses are forwarded to a dedicated Slack channel.

## Tech Stack & Lab Infrastructure
* **CI/CD Orchestration:** Jenkins
* **Cloud Infrastructure:** AWS (ECS, ECR, EC2, IAM)
* **Containerization:** Docker
* **Code Quality & Testing:** SonarQube, Maven, Checkstyle
* **Artifact Management:** Nexus Repository Manager
* **Security & Compliance:** Trivy
* **Monitoring & Alerts:** Slack Notifications
* **Base OS:** Linux (Ubuntu)

## Key Implementation Highlights

### Multi-Stage Docker Optimization
Leveraged multi-stage Docker builds to dramatically reduce final deployment footprints. By separating the Maven build dependency workspace from the lightweight runtime container, the attack surface area and infrastructure resource overhead were minimized.

### Continuous Security & Quality Gates
Enforced automated guardrails inside the execution flow. The pipeline is architected to immediately fail the build stage if SonarQube quality gate thresholds are breached or if Trivy detects critical vulnerability exposures in system layers.

### Automated Cloud Delivery
Created declarative deployment routines using the AWS CLI. The pipeline dynamically updates Amazon ECS task definitions with newly pushed ECR tags, ensuring seamless rolling updates to web applications without manual container configurations.

---
### TOOls Setup
1. Server Provisioning & Configuration

Create 3 servers on aws with the names (Jenkin server , Sonarqube, Server, Nexus server)

Server instance type: c7i-flex.large for all

server setup:

Server 1 - Ubunut latest : Jenkins Server Setup - Refer jenkins_server_setup.docx file

Server 2 - Ubunut latest: SonarQube Server (use sonarqube User Data script from github repo)

Server 3 -  Amazon linux 2023 latest: Nexus Repository Manager (use nexus user data script)

Note: Do the jenkins setup after creating the server, while for user data script use it while creating the ec2 instance.

Security groups setup :

Set Inbound traffic for Jenkins, sonarqube, nexus server

Allow port 8080, 22 from anywhere for jenkins security group.

Allow port 8081, 22 from anywhere for nexus security group.

Allow port 80 ,22 from anywhere for sonarqube security group.

Now, after the above steps validate all the services on the servers are running fine or not.

2. Plugins requirement on Jenkins server.

Plugins: AWS SDK ,ECR, Docker pipeline.

3. Store credentials on Jenkins server .








