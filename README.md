# 🚀 Python App — CI/CD with Jenkins & GitHub 


>Both the Python App — CI/CD with Jenkins & GitHub and the Node.js App — CI/CD with Jenkins & GitHub projects are almost the same, with only slight differences. You can also check my Node.js app project for reference.<br>
>Here, I’ve shared only the quick workflow and changes. For full documentation, check the Node.js app repo.for reference.[GitHub Repo](https://github.com/nikiimisal/Project-Node.js-App-CI-CD-with-Jenkins-GitHub)<br>

>For the Python app setup, I’ve created a repository that includes the Jenkinsfile and other required setup materials.<br>
👉 Click here to view the repo.[GitHub Repo](https://github.com/nikiimisal/python.app-CICD-setup-repo/tree/main)

This project demonstrates a **complete CI/CD pipeline** for a **Python application** using **Jenkins and GitHub Webhooks**, fully automated from **code push → build → test → deploy**.

>>This documentation is beginner-friendly as well as professional.
---

<p align="center">
  <img src="https://github.com/nikiimisal/Project-Python-App-CI-CD-with-Jenkins-GitHub/blob/main/img/uuuuu.png?raw=true" width="700" alt="Python CI/CD Flow Image">
</p>

---

## 🧩 Overview

- Automates deployment of a **Python application** using Jenkins CI/CD.
- Integrates **GitHub Webhooks** to auto-trigger builds when code is pushed.
- Pulls latest code from the GitHub repository.
- Installs all required Python dependencies (`requirements.txt`).
- Runs automated tests (e.g., `pytest`) to ensure code quality.
- Deploys the updated Python app to the target server.
- Fully automated workflow — from code commit to live deployment.

---

## 🧰 Tools & Technologies

- **Jenkins** – CI/CD automation  
- **Python 3 & pip** – Application runtime & dependency manager  
- **GitHub Webhooks** – Build trigger on push  
- **Virtualenv / Gunicorn** – Python process management  
- **EC2 (Ubuntu)** – Deployment environment  

---

## 🔌 Jenkins Plugins (at minimum)

- **Git Plugin** → For repo clone/pull  
- **GitHub Plugin** → Enables webhook triggers  
- **Pipeline Plugin** → Defines multi-stage CI/CD flow  
- **SSH Agent Plugin** → Remote deploys to EC2  
- **Pipeline Stage View Plugin** → Stage-wise visualization 
- **Blue Ocean view**

---

## 📋 Prerequisites

- Python app repo on GitHub  
- Jenkins server (Ubuntu) reachable by GitHub  
- Target EC2 server with SSH access  
- `requirements.txt` for dependencies  
- Simple app entry point (e.g., `app.py`)  

---

## 🔁 High-level CI/CD Flow

1. Developer pushes code to main branch  
2. GitHub Webhook triggers Jenkins  
3. Jenkins pulls repo, installs dependencies, runs tests  
4. If tests pass, deploys new version to EC2  
5. SSH into EC2 → update code → restart Gunicorn/Flask app  
6. Post-deploy check + notifications  

---

## 🪜 Step-by-Step Implementation

### Step 1: Launch EC2s

Create 2 EC2 instances in same VPC:  
- Jenkins Server → Port `8080`  
- Target Server → Ports `22`, `5000`  
<br>
>Optional you can add these steps in jenkins file
Run this command in terminal

```bash
sudo hostnamectl hostname python-app
sudo apt update
sudo apt install python3 python3-pip -y
sudo pip install virtualenv gunicorn
```

---

### Step 2: Create GitHub Repository

Repo Name: `python-app-CICD`  
[GitHub Repo](https://github.com/nikiimisal/python.app-CICD-setup-repo.git)

#### Add Webhook

Payload URL → `http://<JENKINS_PUBLIC_IP>:8080/github-webhook/`

---

### Step 3: Add SSH Credentials in Jenkins

> Manage Jenkins → Credentials → Global → Add Credentials

- ID: `python-app-key`  
- Username: `ubuntu`  
- Paste your EC2 private key  

---

### Step 4: Create Jenkins Pipeline Job

- **Name:** `python-app-deploy-cicd`  
- **Type:** Pipeline  
- **Trigger:** *GitHub hook trigger for GITScm polling*  
- **SCM URL:** `https://github.com/nikiimisal/python-app-CICD.git`  
- **Branch:** `main`  
- **Script Path:** `jenkinsfile`

---

### Step 5: Jenkinsfile Configuration

```groovy
pipeline {
    agent any

    environment {
        SERVER_IP      = '16.171.161.138'
        SSH_CREDENTIAL = 'python-app-key'
        REPO_URL       = 'https://github.com/nikiimisal/python-app-CICD.git'
        BRANCH         = 'main'
        REMOTE_USER    = 'ubuntu'
        REMOTE_PATH    = '/home/ubuntu/python-app'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Upload Files to EC2') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'mkdir -p ${REMOTE_PATH}'
                        scp -o StrictHostKeyChecking=no -r * ${REMOTE_USER}@${SERVER_IP}:${REMOTE_PATH}/
                    '''
                }
            }
        }

        stage('Install Dependencies & Run App') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                            cd ${REMOTE_PATH} &&
                            python3 -m venv venv &&
                            source venv/bin/activate &&
                            pip install -r requirements.txt &&
                            nohup gunicorn --bind 0.0.0.0:5000 app:app --daemon
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Python App Deployed Successfully!'
        }
        failure {
            echo '❌ Deployment Failed. Check Jenkins Logs.'
        }
    }
}
```

> ⚙️ Replace `SERVER_IP`, `SSH_CREDENTIAL`, and paths with your actual setup.

---

### Step 6: Push Code to GitHub

```bash
git init
git add .
git commit -m "Initial Python CI/CD setup"
git push -u origin main
```
>Now we pushed code to GitHub, a webhook instantly notifies the Jenkins server. Jenkins then automatically pulls the latest code, installs dependencies, runs tests, builds the application, and deploys it to the target server

---

### Step 7: Verify Build

Open jenkin:

-- click build know

>If the pipeline runs successfully, that’s great 👍 —<br>
but if it fails, check the Console Output in Jenkins to see the errors and fix them accordingly.
---

### Step 8: Access App in Browser

Open:  
`http://<EC2-Public-IP>:5000`

---

## 🧠 Troubleshooting

| Issue | Solution |
|--------|-----------|
| Webhook not triggering | Ensure Jenkins Public IP accessible |
| Permission denied (SSH) | Check private key & username |
| Jenkins build fails | Review console logs |
| Gunicorn not starting | Check `nohup.out` on EC2 |
| Port not accessible | Open port 5000 in EC2 Security Group |

---

## 🎯 Conclusion

By integrating Jenkins with GitHub Webhooks, this project enables **fully automated Python app deployment** — from commit to live server.

✅ **Key Benefits:**  
- Continuous Integration & Delivery (CI/CD)  
- Automated testing & deployment  
- Fast, reliable updates  
- Zero manual work  

---

**Author:** nikiimisal 😜  
**License:** MIT  
