# Jenkins CI/CD Web Application

## Project Overview

This project is a complete CI/CD pipeline for deploying a web application using Jenkins, GitHub, Docker, Nginx, and AWS EC2.

The pipeline automates the complete application delivery process from source code management to Docker-based deployment.

The workflow is:

GitHub → Jenkins → Build → Test → Artifact → Deploy

---

## Technologies Used

* AWS EC2
* Ubuntu Linux
* Git
* GitHub
* Jenkins
* Jenkins Pipeline
* Docker
* Nginx
* SSH
* GitHub Webhook

---

## Application

The project is a simple HTML web application.

The main application file is:

```text
index.html
```

The application is served using the Nginx web server inside a Docker container.

---

## Project Structure

```text
jenkins-cicd-webapp/
│
├── index.html
├── Dockerfile
├── Jenkinsfile
└── README.md
```

### index.html

Contains the web application interface.

### Dockerfile

Defines how the application is packaged into a Docker image using Nginx.

### Jenkinsfile

Defines the complete CI/CD pipeline.

### README.md

Contains the project documentation.

---

# CI/CD Architecture

```text
                 Developer
                     |
                     | git push
                     v
                GitHub Repository
                     |
                     | Webhook
                     v
              Jenkins Controller
                     |
                     | Pipeline
                     v
              Jenkins Agent
                     |
          +----------+----------+
          |                     |
          v                     v
      Git Checkout          Docker Build
                                |
                                v
                         Docker Image
                                |
                                v
                        Automated Tests
                         /           \
                        /             \
                       v               v
                  HTML Test       Docker Test
                        \             /
                         \           /
                          v         v
                         Artifact
                            |
                            v
                    Environment Deploy
                     /      |       \
                    /       |        \
                  DEV      TEST      PROD
                   |         |         |
                   v         v         v
                Docker    Docker    Docker
               Container Container Container
```

---

# Jenkins Pipeline

The Jenkins pipeline is implemented using a Jenkinsfile.

The pipeline contains the following stages:

```text
Checkout
   ↓
Build
   ↓
Parallel Tests
   ↓
Create Artifact
   ↓
Deploy DEV / TEST / PROD
   ↓
Post Actions
```

---

# 1. Checkout Stage

The Checkout stage retrieves the application source code from GitHub.

The repository is accessed using SSH credentials configured in Jenkins.

```text
GitHub Repository
        ↓
Jenkins Checkout
        ↓
Source Code
```

The pipeline checks out the `main` branch.

---

# 2. Build Stage

The Build stage creates a Docker image for the application.

Docker image name:

```text
jenkins-cicd-webapp
```

Each Jenkins build receives a unique Docker image tag using the Jenkins build number.

Example:

```text
jenkins-cicd-webapp:16
```

The pipeline also creates:

```text
jenkins-cicd-webapp:latest
```

This provides versioned Docker images for different Jenkins builds.

---

# 3. Parallel Tests

The pipeline performs two tests in parallel.

```text
             Parallel Tests
                 /      \
                /        \
               v          v
          HTML Test    Docker Test
```

## HTML Test

The HTML test verifies that:

* `index.html` exists
* The file contains valid HTML content
* The expected application text is present

Commands used include:

```bash
test -f index.html
grep -q "<html" index.html
grep -q "DevOps CI/CD Pipeline" index.html
```

---

## Docker Test

The Docker test validates the Nginx configuration inside the Docker image.

```bash
docker run --rm jenkins-cicd-webapp:${BUILD_NUMBER} nginx -t
```

The test confirms that the Nginx configuration is syntactically correct.

---

# 4. Create Artifact Stage

After successful testing, the pipeline creates a deployment artifact.

The artifact contains:

```text
artifact/
├── index.html
├── Dockerfile
└── build-info.txt
```

The `build-info.txt` file contains information such as:

```text
Application: jenkins-cicd-webapp
Build Number: <BUILD_NUMBER>
Environment: <ENVIRONMENT>
```

The artifact is archived in Jenkins using `archiveArtifacts`.

This allows build artifacts to be associated with specific Jenkins builds.

---

# 5. Environment Selection

The pipeline provides three deployment environments:

```text
DEV
TEST
PROD
```

The environment is selected using a Jenkins build parameter.

```text
ENVIRONMENT
```

The selected environment determines which deployment stage runs.

---

# 6. DEV Deployment

When `DEV` is selected, the application is deployed to the DEV environment.

Container:

```text
jenkins-cicd-webapp-dev
```

Port mapping:

```text
5001:80
```

The application runs on port 80 inside the Docker container and is exposed through port 5001 on the host.

---

# 7. TEST Deployment

When `TEST` is selected, the application is deployed to the TEST environment.

Container:

```text
jenkins-cicd-webapp-test
```

Port mapping:

```text
5002:80
```

---

# 8. PROD Deployment

When `PROD` is selected, the application is deployed to the PROD environment.

Container:

```text
jenkins-cicd-webapp-prod
```

Port mapping:

```text
5003:80
```

---

# Environment Deployment Model

```text
                Jenkins Pipeline
                       |
              Select Environment
                       |
          +------------+------------+
          |            |            |
          v            v            v
         DEV          TEST         PROD
          |            |            |
       Port 5001    Port 5002    Port 5003
          |            |            |
          v            v            v
       Docker        Docker       Docker
      Container     Container    Container
```

Only the selected environment is deployed during a pipeline execution.

---

# GitHub Webhook

GitHub Webhook is used to automatically notify Jenkins when code is pushed to the repository.

Workflow:

```text
Developer
    |
    | git push
    v
GitHub
    |
    | Push Event
    v
Jenkins Webhook
    |
    v
Jenkins Pipeline
```

This allows the CI/CD pipeline to start automatically whenever changes are pushed to GitHub.

---

# Docker Deployment

The application is packaged into a Docker image.

The Docker image uses:

```text
nginx:alpine
```

The application is copied into the Nginx web directory:

```text
/usr/share/nginx/html/index.html
```

The container exposes:

```text
80
```

Docker then maps the container port to the appropriate environment port.

---

# Jenkins Agent

The pipeline runs on a Jenkins Linux agent.

The agent performs:

* Git checkout
* Docker image building
* Automated testing
* Artifact creation
* Docker deployment
* Docker cleanup

This demonstrates the use of Jenkins distributed architecture.

---

# Docker Container Lifecycle

For each deployment, the pipeline removes the existing environment container and starts a new container using the latest build image.

```text
Existing Container
       |
       v
   Remove Old
       |
       v
 Start New Container
       |
       v
 Running Application
```

This ensures that the selected environment runs the Docker image generated by the current Jenkins build.

---

# Docker Image Versioning

Jenkins build numbers are used to identify Docker image versions.

Example:

```text
Build #14
jenkins-cicd-webapp:14

Build #15
jenkins-cicd-webapp:15

Build #16
jenkins-cicd-webapp:16
```

This makes it possible to identify which Jenkins build produced a particular Docker image.

---

# Pipeline Automation

The pipeline automates:

```text
Source Code Checkout
        ↓
Docker Build
        ↓
HTML Validation
        ↓
Docker Validation
        ↓
Artifact Creation
        ↓
Artifact Archiving
        ↓
Environment Deployment
        ↓
Docker Cleanup
```

No manual deployment steps are required after the pipeline is triggered.

---

# Docker Cleanup

After the pipeline completes, unused Docker images are removed using:

```bash
docker image prune -f
```

This helps reduce unnecessary Docker disk usage on the Jenkins agent.

---

# Key Features

## Continuous Integration

Code changes pushed to GitHub can automatically trigger Jenkins.

## Automated Build

Jenkins automatically creates a Docker image from the application.

## Automated Testing

HTML and Docker/Nginx tests run automatically.

## Parallel Testing

Independent tests run simultaneously.

## Artifact Management

Build artifacts are archived in Jenkins.

## Environment-Based Deployment

The pipeline supports:

```text
DEV
TEST
PROD
```

## Containerized Deployment

The web application runs inside Docker containers using Nginx.

## Versioned Builds

Jenkins build numbers are used to version Docker images.

## Pipeline as Code

The entire CI/CD process is defined inside the Jenkinsfile.

---

# Project Workflow

```text
              CODE
                |
                v
             GITHUB
                |
                v
            WEBHOOK
                |
                v
             JENKINS
                |
                v
            CHECKOUT
                |
                v
             BUILD
                |
                v
             TEST
          /         \
         v           v
    HTML TEST    DOCKER TEST
          \         /
           \       /
             v   v
           ARTIFACT
                |
                v
           DEPLOYMENT
          /     |     \
         v      v      v
       DEV    TEST    PROD
         |      |      |
         v      v      v
      DOCKER  DOCKER  DOCKER
       APP     APP     APP
```

---

# Project Outcome

The project successfully demonstrates an end-to-end Jenkins CI/CD workflow for a Dockerized web application.

The application source code is maintained in GitHub.

GitHub Webhook triggers Jenkins automatically after a push.

Jenkins checks out the source code, builds the Docker image, runs automated tests, creates and archives artifacts, and deploys the application to the selected environment.

The project demonstrates the practical integration of:

```text
GitHub
   +
Jenkins
   +
GitHub Webhook
   +
Docker
   +
Nginx
   +
AWS EC2
   =
CI/CD Pipeline
```

---

# Skills Demonstrated

* Linux
* AWS EC2
* Git
* GitHub
* GitHub Webhooks
* Jenkins
* Jenkinsfile
* Jenkins Agents
* Jenkins Credentials
* CI/CD
* Docker
* Nginx
* Automated Testing
* Artifact Management
* Environment-based Deployment
* Pipeline Automation

---

# Conclusion

This project demonstrates how a web application can be automatically built, tested, packaged, and deployed using a CI/CD pipeline.

The complete automated workflow is:

```text
Developer
    ↓
Git Push
    ↓
GitHub
    ↓
Webhook
    ↓
Jenkins
    ↓
Checkout
    ↓
Docker Build
    ↓
Automated Tests
    ↓
Artifact
    ↓
Environment Deployment
    ↓
Docker Container
    ↓
Nginx Web Application
```

The project provides practical hands-on experience with Jenkins CI/CD, GitHub integration, Docker containerization, automated testing, artifact management, and environment-based deployment.
