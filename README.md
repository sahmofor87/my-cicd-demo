# My CI/CD Demo


# My CI/CD Demo


# Prerequisites

Before running this project, ensure the following tools and accounts are installed and configured.

## Required Software

### Git

Verify installation:

```bash
git --version
```

Purpose:

* Source code management
* Push code to GitHub
* Trigger CI/CD pipeline

---

### Node.js and npm

Verify installation:

```bash
node -v
npm -v
```

Purpose:

* Run the application
* Install dependencies
* Execute automated tests

Recommended Version:

```text
Node.js 20.x
```

---

### Docker

Verify installation:

```bash
docker --version
docker ps
```

Purpose:

* Build container images
* Run containers locally
* Push images to JFrog Artifactory

---

### Kubernetes CLI (kubectl)

Verify installation:

```bash
kubectl version --client
```

Purpose:

* Deploy applications
* Manage Kubernetes resources
* Verify deployment status

---

### kind (Kubernetes in Docker)

Verify installation:

```bash
kind version
```

Purpose:

* Local Kubernetes development cluster
* Application deployment testing

Create cluster:

```bash
kind create cluster --name dev-cluster
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
NAME                        STATUS   ROLES           AGE
dev-cluster-control-plane   Ready    control-plane
```

---

### Trivy

Verify installation:

```bash
trivy --version
```

Purpose:

* Container image vulnerability scanning
* Security validation before deployment

Example:

```bash
trivy image my-cicd-demo:latest
```

---

## Required Accounts

### GitHub Account

Required for:

* Source code repository
* GitHub Actions workflows
* Secret management

Repository Example:

```text
https://github.com/<username>/my-cicd-demo
```

---

### JFrog Artifactory Account

Required for:

* Docker image storage
* Private registry access
* Artifact management

Example Registry:

```text
trialgctmla.jfrog.io
```

Verify login:

```bash
docker login trialgctmla.jfrog.io
```

---

## GitHub Repository Secrets

The following secrets must be configured in GitHub:

```text
Repository
└── Settings
    └── Secrets and Variables
        └── Actions
```

Required Secrets:

| Secret Name    | Purpose            |
| -------------- | ------------------ |
| JFROG_USERNAME | JFrog Username     |
| JFROG_TOKEN    | JFrog Access Token |

These secrets are used by GitHub Actions to authenticate with JFrog Artifactory.

---

## Kubernetes Image Pull Secret

Because the Docker image is stored in a private JFrog registry, Kubernetes requires an image pull secret.

Create manually:

```bash
kubectl create secret docker-registry jfrog-regcred \
  --docker-server=trialgctmla.jfrog.io \
  --docker-username=<username> \
  --docker-password=<token>
```

The CI/CD pipeline can also create or update this secret automatically.

---

## Local Environment Validation

Verify all prerequisites:

### Verify Docker

```bash
docker ps
```

### Verify Kubernetes

```bash
kubectl get nodes
```

### Verify kind Cluster

```bash
kind get clusters
```

### Verify JFrog Login

```bash
docker login trialgctmla.jfrog.io
```

### Verify Trivy

```bash
trivy --version
```

### Verify GitHub Access

```bash
git remote -v
```

---

## Expected Environment Architecture

```text
Developer Laptop
│
├── Git
├── Node.js
├── Docker
├── kubectl
├── kind
├── Trivy
│
└── GitHub Repository
      │
      └── GitHub Actions
              │
              ├── Build
              ├── Test
              ├── Trivy Scan
              ├── Push to JFrog
              └── Deploy to Kubernetes

Local kind Cluster
│
└── Application Pods

JFrog Artifactory
│
└── Docker Images
```

## Prerequisite Verification Checklist

* [ ] Git installed
* [ ] Node.js installed
* [ ] npm installed
* [ ] Docker installed and running
* [ ] kubectl installed
* [ ] kind installed
* [ ] Trivy installed
* [ ] GitHub repository created
* [ ] GitHub Actions enabled
* [ ] JFrog Artifactory account configured
* [ ] GitHub secrets configured
* [ ] Kubernetes cluster running
* [ ] Docker image pull secret configured

Once all prerequisites are completed, a simple:

```bash
git push origin main
```

will automatically trigger the entire DevSecOps pipeline.


# CI/CD and DevSecOps Pipeline with GitHub Actions, Trivy, JFrog Artifactory, and Kubernetes

## Project Overview

This project demonstrates a complete end-to-end DevSecOps pipeline using:

* GitHub
* GitHub Actions
* Node.js
* Docker
* Trivy Vulnerability Scanner
* JFrog Artifactory
* Kubernetes
* kind Local Kubernetes Cluster

The pipeline automates the software delivery process from source code commit to Kubernetes deployment while incorporating container security scanning.

## Pipeline Architecture

```text
Developer
    ↓
Git Push
    ↓
GitHub Actions
    ↓
Build & Test
    ↓
Docker Build
    ↓
Trivy Security Scan
    ↓
Push Image to JFrog Artifactory
    ↓
Deploy to Kubernetes
    ↓
Rollout Verification
```

## DevSecOps Workflow

The pipeline is automatically triggered whenever code is pushed to the main branch.

```bash
git add .
git commit -m "Feature update"
git push origin main
```

GitHub Actions then performs all stages automatically.

---

# Stage 1: Source Code Checkout

The pipeline begins by checking out the latest source code from GitHub.

Purpose:

* Retrieve application code
* Ensure pipeline runs against latest commit
* Establish working directory for build process

---

# Stage 2: Application Build and Testing

Dependencies are installed:

```bash
npm install
```

Application tests are executed:

```bash
npm test
```

Purpose:

* Verify application functionality
* Prevent deployment of broken code
* Catch issues early in the pipeline

Benefits:

* Early feedback
* Improved code quality
* Reduced deployment failures

---

# Stage 3: Docker Image Build

After successful testing, a Docker image is created.

Example:

```bash
docker build -t my-cicd-demo .
```

The image is tagged using the Git commit SHA:

```text
trialgctmla.jfrog.io/docker-local/my-cicd-demo:<commit-sha>
```

Additional tag:

```text
latest
```

Benefits:

* Version tracking
* Immutable deployments
* Rollback capability

---

# Stage 4: Trivy Container Security Scan

Before the image is pushed to the registry, Trivy performs a vulnerability scan.

Example:

```bash
trivy image trialgctmla.jfrog.io/docker-local/my-cicd-demo:<commit-sha>
```

Trivy scans:

* Operating System packages
* Application dependencies
* Known CVEs
* Misconfigurations
* Secrets
* Vulnerable libraries

Severity Levels:

* CRITICAL
* HIGH
* MEDIUM
* LOW

Purpose:

* Shift security left
* Detect vulnerabilities before deployment
* Improve container security posture

Example Findings:

```text
CRITICAL: 0
HIGH: 1
MEDIUM: 5
LOW: 12
```

In enterprise environments, the pipeline can be configured to fail if:

```text
CRITICAL > 0
```

or

```text
HIGH > 0
```

This prevents vulnerable containers from reaching production.

---

# Stage 5: Authentication to JFrog Artifactory

GitHub Actions securely authenticates using repository secrets.

Required GitHub Secrets:

```text
JFROG_USERNAME
JFROG_TOKEN
```

Purpose:

* Secure authentication
* Prevent credential exposure
* Support automated image pushes

---

# Stage 6: Push Image to JFrog Artifactory

After passing security scans, the image is pushed to JFrog Artifactory.

Example:

```text
trialgctmla.jfrog.io/docker-local/my-cicd-demo
```

JFrog serves as:

* Private container registry
* Artifact repository
* Image version store
* Deployment source

Benefits:

* Secure image storage
* Version management
* Enterprise artifact governance

---

# Stage 7: Kubernetes Deployment

The pipeline deploys the application to Kubernetes using manifests stored in:

```text
k8s/
```

Deployment Manifest:

```text
k8s/deployment.yaml
```

Service Manifest:

```text
k8s/service.yaml
```

The deployment defines:

* Replica count
* Container image
* Image pull secret
* Container port
* Labels and selectors

The service provides network access to the application.

---

# Stage 8: Kubernetes Image Pull Secret

Since JFrog Artifactory is private, Kubernetes must authenticate before pulling images.

The pipeline creates:

```text
jfrog-regcred
```

Purpose:

* Authenticate Kubernetes to JFrog
* Enable automated image pulls
* Secure registry access

---

# Stage 9: Rolling Deployment

The deployment image is updated automatically.

Example:

```bash
kubectl set image deployment/my-cicd-demo \
my-cicd-demo=trialgctmla.jfrog.io/docker-local/my-cicd-demo:<commit-sha>
```

Benefits:

* Zero manual deployment steps
* Version-controlled releases
* Consistent deployments

---

# Stage 10: Deployment Verification

The pipeline validates the deployment:

```bash
kubectl rollout status deployment/my-cicd-demo
```

Purpose:

* Verify pods start successfully
* Confirm deployment health
* Detect deployment failures

Success Criteria:

```text
deployment "my-cicd-demo" successfully rolled out
```

---

# Kubernetes Validation Commands

Check pods:

```bash
kubectl get pods
```

Check services:

```bash
kubectl get svc
```

Check deployment:

```bash
kubectl get deployments
```

Check rollout:

```bash
kubectl rollout status deployment/my-cicd-demo
```

---

# Accessing the Application

Port forward:

```bash
kubectl port-forward svc/my-cicd-demo-service 3000:3000
```

Open browser:

```text
http://localhost:3000
```

---

# Project Structure

```text
my-cicd-demo/
├── .github/
│   └── workflows/
│       └── cicd-jfrog-k8s.yml
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
├── app.js
├── package.json
├── package-lock.json
├── Dockerfile
├── README.md
│
└── .gitignore
```

---

# Technologies Used

| Technology        | Purpose                          |
| ----------------- | -------------------------------- |
| GitHub            | Source Code Management           |
| GitHub Actions    | CI/CD Automation                 |
| Node.js           | Application Runtime              |
| Docker            | Containerization                 |
| Trivy             | Container Vulnerability Scanning |
| JFrog Artifactory | Private Image Registry           |
| Kubernetes        | Container Orchestration          |
| kind              | Local Kubernetes Cluster         |
| kubectl           | Kubernetes Management            |

---

# DevSecOps Skills Demonstrated

This project demonstrates practical experience with:

* Git-based workflows
* CI/CD pipeline development
* GitHub Actions automation
* Docker image creation
* Container image versioning
* Trivy vulnerability scanning
* DevSecOps security integration
* JFrog Artifactory administration
* Kubernetes deployments
* Kubernetes service management
* Secure registry authentication
* Automated rollout validation
* Infrastructure troubleshooting

---

# End-to-End Pipeline Summary

```text
Developer modifies application
            ↓
Git Commit
            ↓
Git Push
            ↓
GitHub Actions Triggered
            ↓
Checkout Source Code
            ↓
Install Dependencies
            ↓
Run Tests
            ↓
Build Docker Image
            ↓
Trivy Vulnerability Scan
            ↓
Authenticate to JFrog
            ↓
Push Image to Artifactory
            ↓
Deploy to Kubernetes
            ↓
Verify Rollout
            ↓
Application Running Successfully
```

## Final Result

A fully automated DevSecOps pipeline where a simple `git push` performs:

* Automated testing
* Container image build
* Security scanning with Trivy
* Artifact storage in JFrog Artifactory
* Kubernetes deployment
* Rollout verification

with no manual deployment steps required.

<<<<<<< HEAD

=======
>>>>>>> 3434084 (Add full DevSecOps pipeline with Kubernetes deployment and READme file)
