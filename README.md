# 🚀 Jenkins CI/CD Pipeline (Without Docker)

This project demonstrates a **lightweight CI/CD pipeline using Jenkins and GitHub**, deploying a static web application to an **Apache HTTP Server hosted on AWS EC2**.

***

## 🏗️ Architecture Overview

This setup uses **two EC2 instances**:

* **EC2 Instance 1 (CI/CD Server)**  
  Hosts Jenkins and integrates with GitHub.

* **EC2 Instance 2 (Web Server)**  
  Hosts Apache (`httpd`) and serves the application.

***

## 📐 Architecture Diagram

```
                         ┌──────────────────────┐
                         │     Developer        │
                         │  Push Code to GitHub │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │     GitHub Repo      │
                         │   Source Code Store  │
                         └──────────┬──────────┘
                                    │ Webhook Trigger
                                    ▼
┌────────────────────────────────────────────────────────────┐
│               EC2 Instance 1 (Jenkins Server)              │
│----------------------------------------------------------│
│ Jenkins Pipeline                                          │
│  ├── Checkout Code (GitHub)                               │
│  ├── Build Step (optional for static app)                 │
│  ├── Test Step                                            │
│  └── Deploy via SSH (SCP + Commands)                      │
│                                                          │
│ Tools Installed:                                          │
│  - Jenkins                                                │
│  - Git                                                    │
│  - SSH Agent Plugin                                       │
└───────────────┬───────────────────────────────────────────┘
                │ Secure SSH Connection (Key-based)
                ▼
┌────────────────────────────────────────────────────────────┐
│           EC2 Instance 2 (Apache Web Server)              │
│----------------------------------------------------------│
│ Apache HTTP Server (httpd)                                │
│  ├── Receives index.html via SCP                          │
│  ├── Deploys to /var/www/html/                            │
│  └── Serves application on Port 80                        │
└────────────────────────────────────────────────────────────┘
```

***

## 🔄 CI/CD Pipeline Workflow

### 🔹 1. Code Push

* Developer pushes code to GitHub repository
* Jenkins is triggered automatically (via webhook or polling)

***

### 🔹 2. Checkout Stage

```groovy
checkout scm
```

* Jenkins pulls latest code from GitHub repository

***

### 🔹 3. Build Stage

```groovy
echo "Build step..."
```

* Placeholder step
* Can include compilation/minification if needed

***

### 🔹 4. Test Stage

```groovy
echo "Test step..."
```

* Placeholder for:
  * HTML validation
  * Unit tests (if applicable)

***

### 🔹 5. Deploy Stage ✅ (Core Functionality)

Jenkins securely connects to EC2 Instance 2 and deploys files.

#### ✔ Copy File to Server

```bash
scp index.html ec2-user@<EC2-2-IP>:/home/ec2-user/
```

#### ✔ Move to Apache Directory

```bash
sudo cp /home/ec2-user/index.html /var/www/html/index.html
```

#### ✔ Restart Apache Server

```bash
sudo systemctl restart httpd
```

***

## 📜 Jenkins Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any

    environment {
        DEPLOY_SERVER = "ec2-user@172.31.23.4"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Build step..."
            }
        }

        stage('Test') {
            steps {
                echo "Test step..."
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['ec2-target-server-key']) {
                    sh """
                    scp -o StrictHostKeyChecking=no index.html ${DEPLOY_SERVER}:/home/ec2-user/

                    ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} '
                        sudo cp /home/ec2-user/index.html /var/www/html/index.html
                        sudo systemctl restart httpd
                    '
                    """
                }
            }
        }
    }
}
```

***

## ⚙️ Prerequisites

### ✅ EC2 Instance 1 (Jenkins Server)

* Jenkins installed
* Git installed
* SSH Agent Plugin configured
* GitHub webhook configured

***

### ✅ EC2 Instance 2 (Apache Server)

* Install Apache:

```bash
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

***

### ✅ Networking & Security

* Security Group Rules:
  * Port **22** → SSH
  * Port **8080** → Jenkins
  * Port **80** → HTTP (Apache)
* SSH key configured between EC2 instances

***

## 🔐 Credentials Setup

* Add SSH private key in Jenkins:
  * Credential ID: `ec2-target-server-key`
* Used by:

```groovy
sshagent(credentials: ['ec2-target-server-key'])
```

***
