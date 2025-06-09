# Automating Django Deployment with Jenkins: A Beginner-Friendly Guide

This tutorial is based on my self-learning journey with Jenkins and aims to introduce the fundamentals of using Jenkins—a popular continuous integration (CI) tool—for deploying a Django-based web application.

---

## Prerequisites

To follow along, you should have:

- Basic knowledge of **Django** and **Linux**
- A **remote VPS** or server for deployment
- A Git repository containing your Django project

---

## Deployment Overview

1. Create a dedicated Linux user for deployment  
2. Clone the deployment repository  
3. Set up a systemd service to manage the Django app  
4. Create WSGI and helper scripts  
5. Install and configure Jenkins using Docker  
6. Create a `Jenkinsfile` with the deployment pipeline  
7. Run the Jenkins deployment job  
8. Review and debug the output  

---

## 1. Create a Deployment User with Restricted Permissions

Create a non-root user specifically for deploying the app:

```bash
sudo useradd -m deployuser
sudo passwd deployuser
```
Create a group for deployment and configure sudo privileges:
```bash
sudo groupadd deploy
sudo usermod -aG deploy deployuser
sudo visudo
```
Add the following lines to allow restarting services without a password prompt:
```bash
Cmnd_Alias NGINX_RESTART = /usr/sbin/service nginx restart
Cmnd_Alias GUNICORN_RESTART = /usr/sbin/service gunicorn_django_app restart

%deploy ALL=(ALL) NOPASSWD: NGINX_RESTART, GUNICORN_RESTART

```
## 2. Clone the Deployment Repository

Log in as the new user and clone the app repo:
```bash
cd /home/deployuser/
git clone https://your.repo.url.git
```
## 3. Create a systemd Service for the Django App

This allows your app to be managed like a regular Linux service.
Example django_app.service file:

```ini
[Unit]
Description=Django App Service
After=network.target

[Service]
User=deployuser
Group=deploy
WorkingDirectory=/home/deployuser/yourproject
ExecStart=/home/deployuser/yourproject/venv/bin/gunicorn --config /home/deployuser/yourproject/gunicorn_conf.py djangolearning.wsgi
Restart=always

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```bash
sudo systemctl enable django_app
sudo systemctl start django_app
```
## 4. Create WSGI and Helper Bash Scripts

These scripts simplify repetitive deployment tasks.

`checkout.sh`
```bash
#!/bin/bash
cd /home/deployuser/yourproject
git pull
```
`envsetup.sh`
```bash
#!/bin/bash

sudo apt-get install python3-venv -y

if [ ! -d "venv" ]; then
    python3 -m venv venv
fi

source venv/bin/activate

pip install -r requirements.txt
pip install -r requirements-depl.txt

python manage.py makemigrations
python manage.py migrate
```
`gunicorn_conf.py`
```bash
bind = '0.0.0.0:40214'
workers = 2
wsgi_app = "djangolearning.wsgi"
```

## 5. Start and Configure Jenkins (via Docker)

Use Docker Compose to run Jenkins and socat (to allow Jenkins to access Docker API):
```yaml
version: '3.8'

networks:
  jenkins:

services:
  jenkins-master:
    build: .
    image: jenkins/main
    container_name: jenkins-main
    ports:
      - "18080:8080"
      - "50000:50000"
    volumes:
      - ./jenkins-data:/var/jenkins_home
      - ./jenkins-docker-certs:/certs/client:ro
    environment:
      - DOCKER_HOST=tcp://docker:2376
      - DOCKER_CERT_PATH=/certs/client
      - DOCKER_TLS_VERIFY=1
    networks:
      - jenkins

  jenkins-socat:
    image: alpine/socat
    ports:
      - "2376:12375"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: "tcp-listen:12375,fork,reuseaddr unix-connect:/var/run/docker.sock"
    networks:
      - jenkins
```
In Jenkins:
 - Install basic plugins
 - Install SSH and Git plugins
 - Add credentials for Git SCM and SSH access

## 6. Create a Jenkinsfile for the Pipeline

Example Jenkinsfile:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'your-credentials-id', url: 'https://your.repo.url.git'
            }
        }

        stage('Set Up Environment') {
            steps {
                sh './envsetup.sh'
            }
        }

        stage('Restart Service') {
            steps {
                sh './gunicorn.sh'
            }
        }
    }
}
```
## 7. Run the Deployment Job
Trigger the Jenkins pipeline job from the Jenkins dashboard. Monitor logs in
real time to ensure all steps pass correctly.

## 8. Analyze the Output

Use Jenkins build logs to check:
 - If Git checkout was successful
 - Any errors during pip install or migrations
 - Service status via systemctl status django_app

Final Thoughts
For small projects, some steps (like scripting git clone) may seem 
like overkill—but this setup provides a scalable and repeatable deployment process. 
Over time, Jenkins pipelines become invaluable for automated testing and delivery.







