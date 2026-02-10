# 🚀 Kubernetes CI/CD Pipeline with GitHub Actions & ArgoCD

A production-grade CI/CD pipeline implementing GitOps principles for automated containerized application deployment to Kubernetes.


## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Setup Guide](#setup-guide)
- [Pipeline Workflow](#pipeline-workflow)
- [Project Structure](#project-structure)
- [Testing the Pipeline](#testing-the-pipeline)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This project demonstrates a complete CI/CD pipeline that automates the entire software delivery lifecycle from code commit to production deployment. It implements GitOps best practices with clear separation between application code and infrastructure configuration.

### Key Highlights

- ✅ **Zero-touch deployment** from git push to production
- ✅ **GitOps-based** infrastructure management
- ✅ **Automated container** builds and registry management
- ✅ **Self-healing** deployments with ArgoCD
- ✅ **Instant rollbacks** via Git history
- ✅ **Complete audit trail** of all changes

## 🏗️ Architecture

The pipeline follows a GitOps methodology with two separate repositories:

1. **Application Repository** (This repo) - Contains source code and CI pipeline
2. **GitOps Repository** - Contains Kubernetes manifests and CD configuration

### Pipeline Flow

```
Developer commits code
        ↓
GitHub Actions triggers (CI)
        ↓
Build & test application
        ↓
Build Docker image
        ↓
Push to GitHub Container Registry
        ↓
Update GitOps repo with new image tag
        ↓
ArgoCD detects change (CD)
        ↓
Auto-sync to Kubernetes cluster
        ↓
Rolling deployment with new pods
        ↓
Application live in production
```

## ✨ Features

### Continuous Integration (CI)
- Automated testing and code quality checks
- Docker image building and optimization
- Multi-stage builds for smaller images
- Semantic versioning and image tagging
- Push to GitHub Container Registry
- Automated GitOps repository updates

### Continuous Deployment (CD)
- GitOps-based declarative deployments
- Automated synchronization via ArgoCD
- Self-healing and drift detection
- Automated pruning of unused resources
- Health status monitoring
- Rollback capability via Git

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| **CI Platform** | GitHub Actions |
| **CD Platform** | ArgoCD |
| **Container Registry** | GitHub Container Registry (GHCR) |
| **Orchestration** | Kubernetes (Docker Desktop) |
| **Containerization** | Docker |
| **Methodology** | GitOps |
| **Runner** | Self-hosted (Windows) |

## 📦 Prerequisites

Before setting up this pipeline, ensure you have:

- [x] Docker Desktop installed with Kubernetes enabled
- [x] `kubectl` CLI configured
- [x] Git installed
- [x] GitHub account with repository access
- [x] Basic understanding of Kubernetes, Docker, and CI/CD concepts

## 🚀 Setup Guide

### Step 1: Clone the Repository

```bash
git clone https://github.com/YOUR-USERNAME/YOUR-REPO.git
cd YOUR-REPO
```

### Step 2: Configure GitHub Secrets

Add the following secrets to your GitHub repository:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Add these repository secrets:

```
GHCR_TOKEN          # GitHub Container Registry access token
GHCR_USERNAME       # Your GitHub username
GITOPS_TOKEN        # Token for GitOps repository access
GITOPS_REPO         # GitOps repository URL
```

**To create tokens:**
- GHCR_TOKEN: Settings → Developer settings → Personal access tokens → Generate (with `write:packages` scope)
- GITOPS_TOKEN: Settings → Developer settings → Personal access tokens → Generate (with `repo` scope)

### Step 3: Set Up Self-Hosted Runner

1. Navigate to **Settings** → **Actions** → **Runners**
2. Click **New self-hosted runner**
3. Follow the platform-specific instructions (Windows/Linux/macOS)

**For Windows:**
```powershell
# Create directory
mkdir actions-runner; cd actions-runner

# Download runner
Invoke-WebRequest -Uri <DOWNLOAD_URL> -OutFile actions-runner-win.zip

# Extract
Expand-Archive -Path actions-runner-win.zip -DestinationPath .

# Configure
./config.cmd --url https://github.com/YOUR-USERNAME/YOUR-REPO --token YOUR-TOKEN

# Run
./run.cmd
```

### Step 4: Install ArgoCD

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=300s

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### Step 5: Access ArgoCD UI

```bash
# Port forward ArgoCD server
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Access at: https://localhost:8080
# Username: admin
# Password: (from previous step)
```

### Step 6: Configure Container Registry Access

```bash
# Create secret for pulling images from GHCR
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=YOUR-GITHUB-USERNAME \
  --docker-password=YOUR-GITHUB-TOKEN \
  --docker-email=YOUR-EMAIL \
  -n default
```

### Step 7: Deploy ArgoCD Application

```bash
# Apply the ArgoCD application manifest
kubectl apply -f argocd-application.yaml
```

## 🔄 Pipeline Workflow

### CI Pipeline (`.github/workflows/ci.yml`)

Triggered on: `push` to `main` branch

**Steps:**
1. Checkout source code
2. Set up Docker Buildx
3. Login to GitHub Container Registry
4. Build Docker image
5. Tag image with commit SHA
6. Push to GHCR
7. Update GitOps repository with new image tag

### CD Pipeline (ArgoCD)

Continuously monitors GitOps repository

**Automated Actions:**
1. Detect changes in GitOps repo
2. Compare desired state vs. actual cluster state
3. Synchronize resources
4. Perform rolling deployment
5. Health check validation
6. Report sync status

## 📁 Project Structure

```
.
├── .github/
│   └── workflows/
│       └── ci.yml              # GitHub Actions CI pipeline
├── src/                        # Application source code
├── Dockerfile                  # Container image definition
├── k8s/                        # Local Kubernetes manifests (optional)
├── docs/                       # Documentation and diagrams
│   └── architecture-diagram.png
├── README.md                   # This file
└── .gitignore
```

**GitOps Repository Structure:**
```
gitops-repo/
├── deployment.yaml             # Kubernetes deployment
├── service.yaml                # Kubernetes service
├── argocd-application.yaml     # ArgoCD application config
└── README.md
```

## 🧪 Testing the Pipeline

### End-to-End Test

1. **Make a code change:**
```bash
# Edit a file (e.g., update version number)
echo "v1.1.0" > VERSION

# Commit and push
git add .
git commit -m "chore: bump version to 1.1.0"
git push origin main
```

2. **Monitor CI Pipeline:**
   - Go to **Actions** tab in GitHub
   - Watch the workflow execution
   - Verify image pushed to GHCR

3. **Check GitOps Repository:**
   - Verify automatic commit updating image tag

4. **Monitor ArgoCD:**
   - Open ArgoCD UI (https://localhost:8080)
   - Watch application sync status
   - Verify new pods are created

5. **Verify Deployment:**
```bash
# Check pod status
kubectl get pods

# Check deployment
kubectl describe deployment YOUR-APP

# Access application (if applicable)
kubectl port-forward deployment/YOUR-APP 8081:8080
```

## 🔧 Troubleshooting

### Common Issues

#### CI Pipeline Fails

**Issue:** Workflow doesn't trigger
- **Solution:** Check branch name in workflow file matches your push branch
- Verify self-hosted runner is online

**Issue:** Docker build fails
- **Solution:** Verify Dockerfile syntax
- Check Docker is running on self-hosted runner

#### ArgoCD Sync Failures

**Issue:** ImagePullBackOff error
- **Solution:** Verify `ghcr-secret` exists in correct namespace
- Check secret credentials are valid
- Confirm image name and tag are correct

**Issue:** Application shows "OutOfSync"
- **Solution:** Click "Sync" in ArgoCD UI
- Check GitOps repository accessibility
- Verify manifest syntax

#### Authentication Issues

**Issue:** Permission denied pushing to GHCR
- **Solution:** Regenerate GHCR_TOKEN with `write:packages` scope
- Update GitHub secret

**Issue:** ArgoCD can't access GitOps repo
- **Solution:** Check GITOPS_TOKEN permissions
- Verify repository URL is correct

### Useful Commands

```bash
# View ArgoCD application logs
kubectl logs -n argocd deployment/argocd-server

# Check pod logs
kubectl logs <pod-name>

# Describe pod for events
kubectl describe pod <pod-name>

# Force ArgoCD sync
argocd app sync YOUR-APP --force

# View ArgoCD app status
argocd app get YOUR-APP

# Rollback deployment (via kubectl)
kubectl rollout undo deployment/YOUR-APP

# View deployment history
kubectl rollout history deployment/YOUR-APP
```

## 📊 Monitoring & Observability

### View Deployment Status

```bash
# Real-time pod status
kubectl get pods -w

# Deployment rollout status
kubectl rollout status deployment/YOUR-APP

# ArgoCD application health
argocd app get YOUR-APP
```

### Logs

```bash
# Application logs
kubectl logs -f deployment/YOUR-APP

# ArgoCD logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server -f
```

## 🎓 Learning Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [ArgoCD Official Docs](https://argo-cd.readthedocs.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [GitOps Principles](https://opengitops.dev/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 👤 Author

**Your Name**
- GitHub: [@your-username](https://github.com/nathaniel-darko)
- LinkedIn: [Your Name](https://linkedin.com/in/nathaniel-darko)
- Medium: [Your Medium](https://medium.com/@nathanieldarko100/)

## 🙏 Acknowledgments

- ArgoCD Team for the excellent GitOps platform
- GitHub Actions for robust CI/CD capabilities
- Kubernetes community for comprehensive documentation

---


**⭐ If you found this project helpful, please consider giving it a star!**

**💬 Questions? Feel free to open an issue or reach o
