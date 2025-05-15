## Overview

Jenkins is a leading open-source automation server used to build, test, and deploy software. This guide focuses solely on setting up Jenkins itself, and configuring basic jobs below you will find steps for this simple demo 

# Flask Jenkins CI/CD Pipeline

This repository demonstrates a simple Flask application with a fully automated CI/CD pipeline powered by Jenkins and Docker.

## Table of Contents
1. [Project Overview](#project-overview)  
2. [Prerequisites](#prerequisites)  
3. [Project Structure](#project-structure)  
4. [Getting Started](#getting-started)  
   - [Clone the Repository](#clone-the-repository)  
   - [Build and Run Locally](#build-and-run-locally)  
5. [Docker Configuration](#docker-configuration)  
6. [Jenkins CI/CD Pipeline](#jenkins-cicd-pipeline)  
   - [Jenkins Setup](#jenkins-setup)  
   - [Jenkinsfile Explained](#jenkinsfile-explained)  
   - [Credentials Management](#credentials-management)  
   - [Pipeline Triggers](#pipeline-triggers)  
7. [Troubleshooting](#troubleshooting)  
8. [License](#license)  

---

## Project Overview

A minimalist Flask web application that responds with “Hello, Jenkins!”. The focus of this repo is not the app itself, but the end-to-end CI/CD workflow:

- **Docker**: Containerize the Flask app  
- **Jenkins**: Automate build, test, and deployment stages  
- **Docker Hub**: Store the built images  

---

## Prerequisites

- Docker installed on the host machine  
- Jenkins server running on the host (not in Docker)  
- Docker Hub account (or another registry)  
- Jenkins user added to the `docker` group (for local daemon access)  

---

## Project Structure

**flask-jenkins-app/
├── app.py           # Flask application
├── Dockerfile       # Docker build instructions
└── Jenkinsfile      # Jenkins pipeline definition**


# Getting Started

Clone the Repository

`git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO`

Build and Run Locally
1. Build the Docker image

```
docker build -t my-flask-app .
```

2. Run the container 
```
docker run -d -p 5000:5000 --name flask-app my-flask-app
```
3. Visit the app at http://localhost:5000

```python
FROM python:3.11-slim
WORKDIR /app
COPY . /app
RUN pip install flask
EXPOSE 5000
CMD ["python", "app.py"]
```

Base Image: Uses Python 3.11 slim

Dependencies: Installs Flask

Port: Exposes 5000

Entry Point: Runs app.py


## Jenkins CI/CD Pipeline

Jenkins Setup
Add Jenkins user to Docker group (allow Jenkins to use Docker locally):

sudo usermod -aG docker jenkins
sudo systemctl restart jenkins


# Install required plugins:

Git Plugin

Pipeline Plugin

Docker Pipeline Plugin

## Create Credentials in Jenkins UI:

Kind: Username with password

ID: dockerhub-creds

Username: Your Docker Hub username

Password: Your Docker Hub password or access token

Create a Pipeline Job:

Definition: Pipeline script from SCM

SCM: Git

Repository URL: https://github.com/YOUR_USERNAME/YOUR_REPO.git

Branch: */main

Script Path: Jenkinsfile

## Jenkinsfile Explained

```
pipeline {
    agent any
    environment {
        IMAGE_NAME = 'YOUR_USERNAME/flask-jenkins-app'
    }
    stages {
        stage('Clone Repo') {
            steps { 
                git url: 'https://github.com/YOUR_USERNAME/YOUR_REPO.git', branch: 'main' 
            }
        }
        stage('Build Docker Image') {
            steps { 
                sh 'docker build -t $IMAGE_NAME .' 
            }
        }
        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }
        stage('Push Docker Image') {
            steps { 
                sh 'docker push $IMAGE_NAME' 
            }
        }
        stage('Deploy') {
            steps { 
                sh 'docker run -d -p 5000:5000 --name flask-app $IMAGE_NAME' 
            }
        }
    }
    post {
        always { echo 'Pipeline execution complete.' }
    }
}

```

Clone Repo: Retrieves code from GitHub

Build Docker Image: Uses your Dockerfile

Login to Docker Hub: Secure credential injection

Push Docker Image: Publishes to registry

Deploy: Runs the container locally


# Credentials Management

Use withCredentials to avoid exposing secrets in logs

Store credentials under Global domain in Jenkins

# Pipeline Triggers

If your Jenkins host is not publicly reachable, use one of:

Poll SCM: In job config → Build Triggers → Poll SCM → H/5 * * * *

Build periodically: Cron schedule in job config

Manual: Click Build Now in the UI