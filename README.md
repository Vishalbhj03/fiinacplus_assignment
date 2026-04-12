# CI/CD Pipeline for Flask Application using Git, Jenkins, Docker and Kubernetes

## Overview

This project demonstrates a complete CI/CD pipeline for a simple Flask-based application. The goal of this setup is to automate the process of building, packaging, and deploying the application whenever changes are pushed to the GitHub repository.

The pipeline is implemented using Jenkins and is triggered automatically through GitHub webhooks. The application is containerized using Docker and deployed to a Kubernetes cluster (Minikube).

---

## Tech Stack

- Python (Flask)
- Docker
- Jenkins (running on AWS EC2)
- Kubernetes (Minikube)
- GitHub (for source code and webhook triggering)

---

## How the Pipeline Works

Whenever a developer pushes code to the GitHub repository, the following steps are executed automatically:

1. GitHub webhook triggers the Jenkins pipeline  
2. Jenkins pulls the latest code from the repository  
3. A Docker image is built using the application code  
4. The image is tagged with a unique build number and also as `latest`  
5. The image is pushed to DockerHub  
6. Jenkins updates the Kubernetes deployment with the new image  
7. Kubernetes rolls out the updated application  

This ensures that the latest version of the application is always deployed without manual intervention.

---

## Project Structure

- Jenkinsfile → Contains pipeline definition (CI/CD logic)  
- Dockerfile → Used to build the application container  
- requirements.txt → Python dependencies  
- app.py → Flask application code  
- k8s/deployment.yaml → Kubernetes deployment configuration  
- k8s/service.yaml → Kubernetes service configuration  

---

## Setup Instructions

### 1. Clone the Repository

```
git clone https://github.com/Vishalbhj03/two-tier-flask-app.git
cd two-tier-flask-app
```

---

### 2. Jenkins Setup (on AWS EC2)

- Install Jenkins on EC2 instance  
- Install required plugins:  
  - Git Plugin  
  - Docker Pipeline  
  - Kubernetes CLI  
- Ensure Docker is installed and accessible by Jenkins  

---

### 3. Add DockerHub Credentials in Jenkins

- Go to: Manage Jenkins → Credentials  
- Add Username/Password:  
  - ID: docker-creds  
  - Username: your DockerHub username  
  - Password: DockerHub password or access token  

---

### 4. Configure GitHub Webhook

Go to your GitHub repository → Settings → Webhooks → Add webhook  

Payload URL:
```
http://<EC2-PUBLIC-IP>:8080/github-webhook/
```

Content type: application/json  
Trigger: Just the push event  

---

### 5. Kubernetes Setup (Minikube)

```
minikube start --driver=docker
kubectl get nodes
```

Make sure Kubernetes cluster is running before triggering pipeline.

---

### 6. Create Jenkins Pipeline Job

- Create a new Pipeline job  
- Select: Pipeline script from SCM  
- SCM: Git  
- Repository URL: your GitHub repo  
- Branch: master (or main)  
- Script path: Jenkinsfile  
- Enable: GitHub hook trigger for GITScm polling  

---

## Deployment Details

- Docker Image: vishalbhj/flask-two-tier-app  
- Kubernetes Deployment: flask-app  
- Service Type: NodePort  
- Exposed Port: 30007  

---

## Key Features

- Fully automated CI/CD pipeline  
- Webhook-based trigger (no manual builds)  
- Docker-based containerization  
- Kubernetes deployment using Minikube  
- Dynamic image update using `kubectl set image`  
- Automatic rollout verification  

---

## Common Issues

- Ensure port 8080 is open in EC2 security group  
- Docker permissions should be enabled for Jenkins user  
- Kubernetes cluster must be running before deployment  
- DockerHub credentials should be correctly configured  

---




