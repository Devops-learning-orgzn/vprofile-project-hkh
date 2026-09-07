# 🚀 CI/CD Pipeline Setup Guide

This document simplifies the complete infrastructure and setup configuration required for your automated multi-server deployment pipeline.

---

## 🏗️ 1. Architecture Overview (3 Linux Servers)

We use three separate Linux instances to handle specific pipeline actions without performance cross-interference.

| Server | Purpose | Core Requirements |
| :--- | :--- | :--- |
| **Server 1** | **Jenkins Build Machine** | Java (JDK 17 or 21), Maven 3.9.9, Docker Engine, Trivy Scan |
| **Server 2** | **SonarQube Server** | Automated deployment via user data script (Port 9000) |
| **Server 3** | **Nexus Repository** | Automated deployment via user data script (Port 8081) |

---

## 🛠️ 2. Jenkins Server Configuration

Once your Jenkins Linux server is running, you must install specific tools and save access credentials.

### 🔹 Required Plugins
Go to **Manage Jenkins** ➡️ **Plugins** ➡️ **Available Plugins** and search for:
* **Git** & **GitHub Integration** (To fetch source code)
* **Maven Integration** & **Pipeline Maven Integration** (To compile Java code)
* **SonarQube Scanner** (To analyze code quality)
* **Nexus Artifact Uploader** (To save builds to Nexus)
* **Docker Pipeline** (To build and push container images)
* **Slack Notification** (To send real-time alerts)

### 🔹 Required Credentials
Go to **Manage Jenkins** ➡️ **Credentials** ➡️ **System** ➡️ **Global credentials** and register these exact IDs:
* `gitcreds` ➡️ Username and Password/Token for GitHub
* `sonacreds` ➡️ Secret text token from your SonarQube profile
* `nexuscreds` ➡️ Admin username and password for Nexus Repository
* `slackcreds` ➡️ Integration secret token generated from Slack

---

## 🐳 3. Automated Server Scripts (User Data)

Paste these automation bash scripts inside the **User Data** section when launching your cloud instances to install Docker and run services instantly.

### 🔹 SonarQube Server Setup
use sonar-setup.sh file form the userdata folder.

### 🔹 Nexus Server Setup
use nexus-setup.sh file form the userdata folder.

## ☁️ 4. AWS Cloud Infrastructure (Manual Configuration)

Log into your AWS Console and prepare your cloud hosting platform manually with these configurations:

1. **Amazon ECR:** Create a private registry repository to securely host your application's compiled Docker images.
2. **Amazon ECS Setup:**
   * Create an **ECS Cluster** (Logical grouping of tasks).
   * Create an **ECS Task Definition** (Specifies blueprints like ECR image path, CPU, and Memory).
   * Create an **ECS Service** (Runs and auto-scales instances of your task definition safely inside the cluster).

---

## 💬 5. Slack Integration

To keep your team updated on pipeline success or failure rates:
1. Create a Slack account and set up a dedicated operations channel.
2. Visit the **Slack App Directory**, search for **Jenkins CI**, and add it to your team workspace.
3. Save the generated **Integration Token Key**.
4. Link this token inside your **Jenkins System Configuration** to allow automated build alerts.
