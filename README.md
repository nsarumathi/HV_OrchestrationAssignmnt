# 🚀 StreamingApp – Production-Grade MERN Stack Deployment on AWS EKS

A fully production-ready **microservices-based MERN application** deployed on **Amazon EKS** using modern DevOps and Cloud Native practices.

---
> 📌 Complete deployment documentation and deployment walkthrough video are included in this repository. Please review them to verify the full end-to-end deployment process and infrastructure setup.
  - Documentation: MERN Architecture.doc
  - Video: deployment_videos folder
---
# 📌 Project Overview

StreamingApp is a scalable video streaming platform that supports:

- 🎬 Premium video streaming
- 👥 Live watch parties
- 💬 Real-time chat system
- 🔐 JWT-based authentication
- 🛠️ Admin asset management
- ☁️ Kubernetes-based deployment

The entire platform is deployed using:

- Kubernetes (EKS)
- Helm Charts
- Traefik Ingress Controller
- Cert-Manager (Let's Encrypt SSL)
- Fluent-Bit + CloudWatch Logging
- AWS SNS + Slack ChatOps
- Jenkins CI/CD Pipeline

---

# 🏗️ System Architecture

## Infrastructure Components

| Component | Purpose |
|---|---|
| Kubernetes (EKS) | Container orchestration |
| Helm | Kubernetes package management |
| Traefik | Ingress controller & routing |
| Cert-Manager | Automated SSL/TLS certificates |
| Fluent-Bit | Cluster log forwarding |
| Amazon CloudWatch | Monitoring & log aggregation |
| AWS SNS | Alerting system |
| AWS Lambda | Slack notification integration |
| Jenkins | CI/CD automation |
| Amazon ECR | Docker image registry |
| Amazon S3 | Video & thumbnail storage |

---

# 🧩 Microservices Architecture

| Service | Port | Description |
|---|---|---|
| authService | 3001 | Authentication & JWT |
| streamingService | 3002 | Video streaming APIs |
| adminService | 3003 | Admin dashboard & uploads |
| chatService | 3004 | Live chat & WebSocket |
| frontend | 3000 | React SPA frontend |

---

# ⚙️ Deployment Workflow

# 1️⃣ AWS S3 & ECR Setup

## Create S3 Bucket

Upload:
- `videos/`
- `thumbnails/`

## Create ECR Repositories

Create repositories for:
- authService
- streamingService
- adminService
- chatService
- frontend

---

# 2️⃣ Jenkins Setup in EC2

## Install Java, Docker & Jenkins

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre

java -version

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins
```

## Enable Jenkins

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

## Unlock Jenkins

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

# 3️⃣ Configure AWS CLI for ECR Access

```bash
sudo apt update
sudo apt install awscli

aws configure
```

---

# 4️⃣ Configure Jenkins Pipeline

## Jenkins Configuration

Configure:
- GitHub webhook
- Credentials
- Pipeline script

## Required GitHub Secrets

- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- CLIENT_URLS
- STREAMING_PUBLIC_URL

---

# 5️⃣ Containerization with Jenkins

## Steps

- Configure `docker-compose.yml`
- Build Docker images
- Push images to Amazon ECR
- Automate deployments via Jenkins pipeline

## IAM Permissions Required

Attach IAM role to EC2 instance with access to:
- ECR
- EKS
- CloudWatch

---

# 6️⃣ Kubernetes Setup

## Install kubectl

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.29.0/2024-01-04/bin/linux/amd64/kubectl

chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin

kubectl version --client
```

## Install eksctl

```bash
curl --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz

sudo mv eksctl /usr/local/bin

eksctl version
```

---

# 7️⃣ Create Amazon EKS Cluster

## Cluster Creation

```bash
eksctl create cluster \
--name mern_cluster \
--region ap-south-2
```

## Configure kubeconfig

```bash
aws eks update-kubeconfig \
--region ap-south-2 \
--name mern_cluster

kubectl get nodes
```

---

# 8️⃣ Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

helm version

helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

---

# 9️⃣ Install Traefik Ingress Controller

## Install Traefik

```bash
helm repo add traefik https://traefik.github.io/charts

helm repo update

helm install traefik traefik/traefik \
--namespace kube-system
```

## Verify Installation

```bash
kubectl get pods -n kube-system

kubectl get svc -n kube-system
```

## Expose Load Balancer

```bash
kubectl patch svc traefik -n kube-system \
-p '{"metadata":{"annotations":{"service.beta.kubernetes.io/aws-load-balancer-scheme":"internet-facing"}}}'
```

---

# 🔟 Deploy Application with Helm

## Create Helm Structure

```bash
mkdir helm_deployment

cd helm_deployment

kubectl create namespace streaming

helm create mern-app-chart
```

## Modify Templates

Add:
- Backend deployment YAMLs
- Frontend deployment YAML
- ConfigMap
- Secret
- Ingress

## Validate Chart

```bash
helm lint .
```

## Install Chart

```bash
helm install mern-release . -n streaming
```

## Verify Deployment

```bash
helm list -A

kubectl get all -n streaming

kubectl get ingress -n streaming
```

---

# 🌐 Internal Cluster Connectivity Testing

```bash
kubectl run test --rm -it --image=curlimages/curl -- sh
```

## Test Services

```bash
curl http://traefik.kube-system.svc.cluster.local

curl http://adminservice.mern-app.svc.cluster.local

curl http://authservice.mern-app.svc.cluster.local

curl http://chatservice.mern-app.svc.cluster.local

curl http://streamingservice.mern-app.svc.cluster.local/api/streaming/videos
```

---

# 🌍 Domain Configuration

Update your `values.yaml`:

```yaml
host: travelmemoryapp.online
```

Apply upgrade:

```bash
helm upgrade mern-release . -n streaming
```

---

# 🔐 SSL/TLS with Cert-Manager

## Install Cert-Manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.crds.yaml

helm repo add jetstack https://charts.jetstack.io

helm repo update

helm install cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace
```

## Verify

```bash
kubectl get pods -n cert-manager

kubectl describe certificate travelmemoryapp-tls -n streaming

kubectl get challenge -A
```

## Create ClusterIssuer

```bash
kubectl apply -f letsencrypt-prod.yaml
```

## Update Ingress

```bash
helm upgrade mern-release . -n streaming
```

---

# 📊 Monitoring & Logging

## Install Fluent-Bit

```bash
kubectl create namespace amazon-cloudwatch

helm upgrade --install fluent-bit eks/aws-for-fluent-bit \
-n amazon-cloudwatch \
--set cloudWatch.region=ap-south-2
```

## Verify

```bash
kubectl get pods -n amazon-cloudwatch
```

## CloudWatch Features

- Centralized logs
- Cluster monitoring
- Load Balancer metrics
- EC2 monitoring dashboards

---

# 🔔 ChatOps Integration (AWS SNS → Slack)

## Alert Flow

```text
CloudWatch Alarm
        ↓
      SNS Topic
        ↓
 Lambda Function
        ↓
 Slack Channel
```

---

# Slack Configuration

## Create

- Slack Workspace
- Slack Channel
- Slack App

## Enable

- Incoming Webhooks

## Configure

- Webhook URL
- SNS Topic Subscription

---

# 🐍 Lambda Function for Slack Alerts

## Responsibilities

- Receives SNS notifications
- Sends formatted alerts to Slack
- Provides deployment & monitoring notifications

---

# 📧 Alert Verification

## Email Alerts

```text
Alarm → SNS → Email
```

## Slack Alerts

```text
Alarm → SNS → Lambda → Slack
```

---

# 📁 Project Structure

```text
helm_deployment/
│
├── mern-app-chart/
│   ├── templates/
│   ├── values.yaml
│   ├── Chart.yaml
│
├── Jenkinsfile
├── docker-compose.yml
```

---

# 🚀 DevOps Features Implemented

- ✅ CI/CD with Jenkins
- ✅ Dockerized Microservices
- ✅ Kubernetes Orchestration
- ✅ Helm-Based Deployment
- ✅ HTTPS with Let's Encrypt
- ✅ Ingress Routing with Traefik
- ✅ Centralized Logging
- ✅ Cloud Monitoring
- ✅ Slack ChatOps Alerts

---

# 🛠️ Tech Stack

## Backend

- Node.js
- Express.js
- MongoDB
- WebSocket

## Frontend

- React.js

## DevOps & Cloud

- Docker
- Kubernetes
- Amazon EKS
- Helm
- Jenkins
- Traefik
- Cert-Manager
- Fluent-Bit
- Amazon CloudWatch
- AWS SNS
- AWS Lambda

---

# 📌 Future Enhancements

- Horizontal Pod Autoscaling
- GitOps with ArgoCD
- Service Mesh (Istio)
- Redis Caching
- Blue-Green Deployment
- Prometheus + Grafana Monitoring

---

# 👨‍💻 Author

**Sharu**

Cloud & DevOps Engineer  
MERN Stack | Kubernetes | AWS | CI/CD | DevOps Automation
