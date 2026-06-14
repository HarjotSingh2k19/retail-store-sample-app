x# 🏪 Zero-Touch GitOps Factory — Retail Store

> **A production-grade, fully automated GitOps pipeline for a polyglot microservices e-commerce application.**
> Built entirely from scratch — every Dockerfile, Terraform file, Ansible playbook, Helm chart, and Jenkinsfile written by hand.

![Architecture](https://img.shields.io/badge/Architecture-Microservices-blue)
![CI](https://img.shields.io/badge/CI-Jenkins-red)
![CD](https://img.shields.io/badge/CD-ArgoCD-orange)
![IaC](https://img.shields.io/badge/IaC-Terraform-purple)
![K8s](https://img.shields.io/badge/Kubernetes-KIND-blue)
![Monitoring](https://img.shields.io/badge/Monitoring-Prometheus%20%2B%20Grafana-orange)

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Repository Structure](#-repository-structure)
- [Services](#-services)
- [Prerequisites](#-prerequisites)
- [Phase 1 — Terraform (Infrastructure)](#-phase-1--terraform-infrastructure)
- [Phase 2 — Ansible (Configuration)](#-phase-2--ansible-configuration)
- [Phase 3 — Containerisation](#-phase-3--containerisation)
- [Phase 4 — Kubernetes + Helm](#-phase-4--kubernetes--helm)
- [Phase 5 — Jenkins CI Pipeline](#-phase-5--jenkins-ci-pipeline)
- [Phase 6 — ArgoCD GitOps](#-phase-6--argocd-gitops)
- [Phase 7 — Observability](#-phase-7--observability)
- [The GitOps Loop](#-the-gitops-loop)
- [Interview Q&A](#-interview-qa)

---

## 🎯 Project Overview

This project deploys a **5-service polyglot microservices application** (Java, Go, Node.js) on Kubernetes using a fully automated DevOps pipeline. A single `git push` triggers the entire pipeline — from building Docker images to deploying on Kubernetes — with **zero manual intervention**.

### What makes this project special?

```
❌ NOT a tutorial copy-paste project
✅ Every single file written from scratch
✅ Production-grade patterns (secrets management, probes, resource limits)
✅ Full GitOps with two-repo pattern
✅ Polyglot microservices (Java Spring Boot, Go, Node.js/NestJS)
✅ Complete observability stack
```

---

## 🏗️ Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        AWS EC2 (t3.large)                       │
│                                                                 │
│  ┌─────────────────┐     ┌───────────────────────────────────┐  │
│  │   Jenkins CI    │     │      KIND Kubernetes Cluster      │  │
│  │   Container     │     │                                   │  │
│  │                 │     │  ┌─────────┐  ┌──────────────┐   │  │
│  │  ┌───────────┐  │     │  │  argocd │  │  monitoring  │   │  │
│  │  │Jenkinsfile│  │     │  │namespace│  │  namespace   │   │  │
│  │  │ 5 stages  │  │     │  └─────────┘  └──────────────┘   │  │
│  │  └───────────┘  │     │                                   │  │
│  │                 │     │  ┌──────────────────────────────┐ │  │
│  │  Port: 8080     │     │  │       retail namespace       │ │  │
│  └────────┬────────┘     │  │  ┌──┐ ┌───────┐ ┌──────┐   │ │  │
│           │              │  │  │ui│ │catalog│ │ cart │   │ │  │
│           │              │  │  └──┘ └───────┘ └──────┘   │ │  │
│           │              │  │  ┌──────┐ ┌──────────┐      │ │  │
│           │              │  │  │orders│ │ checkout │      │ │  │
│           │              │  │  └──────┘ └──────────┘      │ │  │
│           │              │  └──────────────────────────────┘ │  │
│           │              │                                   │  │
│           │              │  NGINX Ingress → Port 80          │  │
│           │              └───────────────────────────────────┘  │
└───────────┼─────────────────────────────────────────────────────┘
            │
            ▼
        DockerHub
```

### GitOps Flow

```
Developer                 GitHub                Jenkins              DockerHub
    │                       │                      │                     │
    │──── git push ────────►│                      │                     │
    │                       │──── webhook ─────────►│                     │
    │                       │                      │── build images ─────►│
    │                       │                      │── push :tag ────────►│
    │                       │                      │                     │
    │                  retail-store-gitops          │                     │
    │                       │◄─── update ──────────│                     │
    │                       │    values.yaml        │                     │
    │                       │                      │                     │
    │                    ArgoCD                    KIND Cluster           │
    │                       │── detects change ───►│                     │
    │                       │                      │◄─── pull image ─────│
    │                       │                      │── rolling update     │
    │                       │                      │── readinessProbe ✅  │
    │                       │                      │── LIVE 🎉            │
```

### Service Communication

```
Browser
   │
   ▼ HTTP :80
NGINX Ingress
   │
   ▼
UI Service (Java Spring Boot :8080)
   ├──► catalog-service:8080  (Go)
   ├──► cart-service:8080     (Java Spring Boot)
   ├──► checkout-service:8080 (Node.js / NestJS)
   │         └──► orders-service:8080
   └──► orders-service:8080   (Java Spring Boot)

All service-to-service communication: ClusterIP (internal only)
External access: Only via NGINX Ingress on port 80
```

---

## 🛠️ Tech Stack

| Category | Tool | Purpose |
|---|---|---|
| Cloud | AWS EC2 | Single compute node |
| IaC | Terraform | Provision VPC, Subnet, SG, EC2 |
| Config Mgmt | Ansible | Install Docker, KIND, kubectl, Helm, Jenkins |
| Containers | Docker | Multi-stage builds for all 5 services |
| Orchestration | Kubernetes (KIND) | Container orchestration |
| Package Mgmt | Helm | Kubernetes manifest templating |
| CI | Jenkins | Build, test, push Docker images |
| CD | ArgoCD | GitOps continuous deployment |
| Registry | DockerHub | Docker image storage |
| Ingress | NGINX | Route external traffic into cluster |
| Monitoring | Prometheus | Metrics collection (pull-based) |
| Dashboards | Grafana | Metrics visualisation |
| State | AWS S3 | Terraform remote state |

---

## 📁 Repository Structure

```
retail-store-sample-app/          ← THIS REPO (App Source)
├── src/
│   ├── ui/                       ← Java Spring Boot (Frontend)
│   │   ├── Dockerfile            ← Multi-stage: Maven → JRE Alpine
│   │   └── .dockerignore
│   ├── catalog/                  ← Go (Product listings)
│   │   ├── Dockerfile            ← Multi-stage: Go build → Debian slim
│   │   └── .dockerignore
│   ├── cart/                     ← Java Spring Boot (Shopping cart)
│   │   ├── Dockerfile
│   │   └── .dockerignore
│   ├── orders/                   ← Java Spring Boot (Order management)
│   │   ├── Dockerfile
│   │   └── .dockerignore
│   └── checkout/                 ← Node.js / NestJS (Checkout flow)
│       ├── Dockerfile            ← Multi-stage: Node build → Node Alpine
│       └── .dockerignore
├── devops/
│   ├── terraform/                ← Infrastructure as Code
│   │   ├── main.tf               ← EC2 instance
│   │   ├── network.tf            ← VPC, Subnet, IGW, Route Table, SG
│   │   ├── variables.tf          ← All input variables
│   │   ├── outputs.tf            ← EC2 IP, SSH command
│   │   ├── terraform.tf          ← Backend (S3 state) + providers
│   │   └── provider.tf           ← AWS provider config
│   └── ansible/
│       ├── ansible.cfg           ← SSH config, key path
│       ├── inventory.ini         ← EC2 IP (gitignored)
│       └── setup-server.yml      ← Master playbook (6 task groups)
├── docker-compose.yml            ← Local validation (all 5 services)
├── kind-config.yaml              ← KIND cluster config (port mappings)
├── Jenkinsfile                   ← 5-stage CI pipeline
└── .gitignore
```

> **Repo 2:** [retail-store-gitops](https://github.com/HarjotSingh2k19/retail-store-gitops) — Contains Helm charts. ArgoCD watches this repo.

---

## 🚀 Services

| Service | Language | Framework | Port | Health Endpoint |
|---|---|---|---|---|
| ui | Java 21 | Spring Boot | 8080 | /actuator/health |
| catalog | Go 1.23 | Gin | 8080 | /catalog/size |
| cart | Java 21 | Spring Boot | 8080 | /actuator/health |
| orders | Java 21 | Spring Boot | 8080 | /actuator/health |
| checkout | Node.js | NestJS | 8080 | /health |

All services use **in-memory persistence** — no database required for this setup.

---

## ✅ Prerequisites

Before you begin, ensure you have:

| Tool | Version | Install |
|---|---|---|
| AWS Account | — | [aws.amazon.com](https://aws.amazon.com) |
| AWS CLI | >= 2.0 | `brew install awscli` |
| Terraform | >= 1.5 | `brew install terraform` |
| Ansible | >= 2.15 | `pip install ansible` |
| Docker Desktop | >= 4.0 | [docker.com](https://docker.com) |
| Git | >= 2.0 | `brew install git` |
| DockerHub Account | — | [hub.docker.com](https://hub.docker.com) |
| GitHub Account | — | [github.com](https://github.com) |

---

## 🟢 Phase 1 — Terraform (Infrastructure)

**Goal:** Provision AWS infrastructure with zero Console clicks.

### Setup AWS Credentials

```bash
aws configure
# AWS Access Key ID: <your-access-key>
# AWS Secret Access Key: <your-secret-key>
# Default region: ap-south-1
# Default output format: json

# Verify
aws sts get-caller-identity
```

### Create S3 Bucket for Remote State

```bash
aws s3 mb s3://gitops-factory-tfstate-<your-account-id> --region ap-south-1
aws s3api put-bucket-versioning \
  --bucket gitops-factory-tfstate-<your-account-id> \
  --versioning-configuration Status=Enabled
```

### Create EC2 Key Pair

```bash
aws ec2 create-key-pair \
  --key-name gitops-factory-key \
  --region ap-south-1 \
  --query 'KeyMaterial' \
  --output text > ~/.ssh/gitops-factory-key.pem

chmod 400 ~/.ssh/gitops-factory-key.pem
```

### Configure Variables

```bash
# Get your current IP
curl -s https://checkip.amazonaws.com

# Create terraform.tfvars (gitignored — never committed)
cat > devops/terraform/terraform.tfvars << 'EOF'
key_name     = "gitops-factory-key"
project_name = "gitops-factory"
my_ip        = "YOUR_IP_HERE/32"
EOF
```

> **Update `backend.tf`** — replace bucket name with yours.

### Run Terraform

```bash
cd devops/terraform

terraform init      # Connect to S3 backend, download providers
terraform fmt       # Auto-format code
terraform validate  # Check syntax
terraform plan      # Preview changes
terraform apply     # Create infrastructure (type 'yes')
```

**Output:**
```
ec2_public_ip  = "xx.xx.xx.xx"
ssh_command    = "ssh -i ~/.ssh/gitops-factory-key.pem ubuntu@xx.xx.xx.xx"
```

### What Gets Created

```
AWS Resources:
├── VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24) in ap-south-1a
├── Internet Gateway
├── Route Table (0.0.0.0/0 → IGW)
├── Security Group
│   ├── 22   → your IP only (SSH)
│   ├── 80   → 0.0.0.0/0 (HTTP)
│   ├── 443  → 0.0.0.0/0 (HTTPS)
│   ├── 8080 → 0.0.0.0/0 (Jenkins)
│   ├── 9090 → 0.0.0.0/0 (Prometheus)
│   └── 30000-32767 → 0.0.0.0/0 (K8s NodePorts)
└── EC2 Instance (t3.large, Ubuntu 24.04, 30GB gp3)
```

> ⚠️ **IP Changes:** Every time EC2 restarts, its public IP changes.
> Update `terraform.tfvars` with new IP and run `terraform apply` to update the SSH rule.

---

## 🟡 Phase 2 — Ansible (Configuration)

**Goal:** Install all required software on EC2 with one command.

### Update Inventory

```bash
# Create inventory.ini with EC2 public IP (gitignored)
cat > devops/ansible/inventory.ini << 'EOF'
[servers]
EC2_PUBLIC_IP_HERE
EOF
```

### Run Ansible Playbook

```bash
cd devops/ansible
ansible-playbook setup-server.yml
```

### What Gets Installed

```
Group 1 — System prep:
  apt update, upgrade, curl, git, unzip, wget

Group 2 — Docker Engine:
  docker-ce, docker-ce-cli, containerd.io
  ubuntu user added to docker group

Group 3 — KIND:
  kind v0.22.0 → /usr/local/bin/kind

Group 4 — kubectl:
  kubectl v1.29.0 → /usr/local/bin/kubectl

Group 5 — Helm:
  helm v3.x → /usr/local/bin/helm

Group 6 — Jenkins:
  docker run jenkins/jenkins:lts
  Ports: 8080 (UI), 50000 (agents)
  Volumes: docker.sock mounted (build images from Jenkins)
```

### Verify Installation

```bash
ssh -i ~/.ssh/gitops-factory-key.pem ubuntu@EC2_IP

docker --version    # Docker version 29.x
kind --version      # kind version 0.22.0
kubectl version     # v1.29.0
helm version        # v3.x
curl http://localhost:8080  # Jenkins HTML
```

---

## 🟠 Phase 3 — Containerisation

**Goal:** Write Dockerfiles from scratch and validate locally.

### Multi-Stage Build Strategy

| Service | Stage 1 (Build) | Stage 2 (Run) | Final Size |
|---|---|---|---|
| ui, cart, orders | Maven + JDK 21 | JRE 21 Alpine | ~180MB |
| catalog | golang:1.23 | debian:bookworm-slim | ~80MB |
| checkout | node:20-alpine | node:20-alpine | ~120MB |

### Local Validation with Docker Compose

```bash
# Build and start all 5 services
docker compose up --build

# Test in browser
open http://localhost:8080
```

**Service ports (local only):**
```
http://localhost:8080  → ui
http://localhost:8081  → catalog
http://localhost:8082  → cart
http://localhost:8083  → orders
http://localhost:8085  → checkout
```

### Build & Push Multi-Platform Images

> ⚠️ **Important:** If building on Apple Silicon (M1/M2), use multi-platform builds
> so images run on AMD64 EC2 instances.

```bash
# Create multi-platform builder
docker buildx create --name multiplatform --use
docker buildx inspect --bootstrap

# Build and push all services
for service in ui catalog cart orders checkout; do
  docker buildx build \
    --platform linux/amd64,linux/arm64 \
    -t YOUR_DOCKERHUB_USER/retail-store-${service}:v1 \
    --push src/${service}/
done
```

---

## 🔵 Phase 4 — Kubernetes + Helm

**Goal:** Bootstrap KIND cluster and deploy via Helm charts.

### Step 1 — Copy kind-config.yaml to EC2

```bash
scp -i ~/.ssh/gitops-factory-key.pem \
  kind-config.yaml \
  ubuntu@EC2_IP:~/kind-config.yaml
```

### Step 2 — Create KIND Cluster

```bash
ssh -i ~/.ssh/gitops-factory-key.pem ubuntu@EC2_IP

kind create cluster --config kind-config.yaml
kubectl cluster-info --context kind-gitops-factory
kubectl get nodes
```

### Step 3 — Install NGINX Ingress Controller

```bash
# Must use KIND-specific manifest
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# Wait for controller to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

> ⚠️ **Critical:** Use the KIND-specific manifest. Using cloud or baremetal
> manifests is the most common silent failure in KIND setups.

### Step 4 — Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD server
kubectl wait --namespace argocd \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/name=argocd-server \
  --timeout=180s

# Patch to NodePort for access
kubectl patch svc argocd-server -n argocd \
  -p '{"spec":{"type":"NodePort"}}'

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

### Step 5 — Deploy Application via ArgoCD

```bash
# Copy argocd-app.yaml to EC2
scp -i ~/.ssh/gitops-factory-key.pem \
  argocd-app.yaml ubuntu@EC2_IP:~/

# Apply on EC2
kubectl apply -f argocd-app.yaml

# Watch pods deploy
kubectl get pods -n retail -w
```

### Step 6 — Access the Application

```
http://EC2_PUBLIC_IP
```

---

## 🔴 Phase 5 — Jenkins CI Pipeline

**Goal:** Automate build → push → GitOps update on every git push.

### First-Time Jenkins Setup

1. Get unlock password:
```bash
ssh -i ~/.ssh/gitops-factory-key.pem ubuntu@EC2_IP \
  "docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword"
```

2. Open Jenkins: `http://EC2_IP:8080`
3. Install suggested plugins + **Docker Pipeline** + **GitHub Integration**
4. Create admin user

### Configure Credentials

Go to: **Manage Jenkins → Credentials → Global → Add Credentials**

| ID | Kind | Value |
|---|---|---|
| `DOCKERHUB_CREDS` | Username with password | DockerHub username + PAT |
| `GITHUB_TOKEN` | Secret text | GitHub Personal Access Token (repo scope) |

### Configure Pipeline Job

1. **New Item** → `retail-store-pipeline` → **Pipeline**
2. **Build Triggers** → ✅ GitHub hook trigger for GITScm polling
3. **Pipeline** section:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: `https://github.com/YOUR_USER/retail-store-sample-app.git`
   - Branch: `*/gitops`
   - Script Path: `Jenkinsfile`

### Pipeline Stages

```
Stage 1 — Checkout
  checkout scm → pulls latest code

Stage 2 — Build Images
  docker build all 5 services
  tagged with $BUILD_NUMBER

Stage 3 — Push to DockerHub
  withCredentials (DOCKERHUB_CREDS)
  echo $DH_PASS | docker login --password-stdin
  docker push all 5 images

Stage 4 — Update GitOps Repo
  withCredentials (GITHUB_TOKEN)
  git clone retail-store-gitops
  sed update all tag values in values.yaml
  git commit -m "ci: bump tags to $BUILD_NUMBER [skip ci]"
  git push

Stage 5 — Done
  echo confirmation message
```

### Configure GitHub Webhook

Go to: `https://github.com/YOUR_USER/retail-store-sample-app/settings/hooks`

```
Payload URL:  http://EC2_IP:8080/github-webhook/
Content type: application/json
Events:       Just the push event
Active:       ✅
```

---

## 🟣 Phase 6 — ArgoCD GitOps

**Goal:** Automatic deployment on every values.yaml change.

### Access ArgoCD UI

```bash
# SSH tunnel from your Mac
ssh -i ~/.ssh/gitops-factory-key.pem \
  -L 8888:localhost:8888 \
  ubuntu@EC2_IP \
  "kubectl port-forward svc/argocd-server 8888:80 -n argocd --address 127.0.0.1"

# Open in browser
open http://localhost:8888
# Username: admin
# Password: (from Phase 4 Step 4)
```

### ArgoCD Application Config

The `argocd-app.yaml` configures:

```yaml
syncPolicy:
  automated:
    prune: true      # Delete resources removed from Git
    selfHeal: true   # Revert manual kubectl changes
  syncOptions:
    - CreateNamespace=true
```

| Setting | Effect |
|---|---|
| `automated` | Sync automatically when Git changes |
| `prune: true` | Delete K8s resources not in Git |
| `selfHeal: true` | Revert manual kubectl changes |
| `CreateNamespace=true` | Auto-create `retail` namespace |

---

## ⚪ Phase 7 — Observability

**Goal:** Monitor cluster health with Prometheus + Grafana.

### Install kube-prometheus-stack

```bash
# On EC2
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --create-namespace \
  --set grafana.adminPassword=admin123 \
  --set prometheus.prometheusSpec.retention=24h \
  --set alertmanager.enabled=false
```

### Access Grafana

```bash
# SSH tunnel from your Mac
ssh -i ~/.ssh/gitops-factory-key.pem \
  -L 3000:localhost:3000 \
  ubuntu@EC2_IP \
  "kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring --address 127.0.0.1"

# Open in browser
open http://localhost:3000
# Username: admin
# Password: admin123
```

### Import Dashboards

| Dashboard ID | Name | Shows |
|---|---|---|
| 6417 | Kubernetes Cluster Monitoring | Pod CPU, memory, network |
| 1860 | Node Exporter Full | EC2 node metrics |

Go to: **Dashboards → New → Import → Enter ID → Load → Select Prometheus → Import**

### Access Prometheus

```bash
ssh -i ~/.ssh/gitops-factory-key.pem \
  -L 9090:localhost:9090 \
  ubuntu@EC2_IP \
  "kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring --address 127.0.0.1"

open http://localhost:9090
```

**Useful PromQL queries:**
```promql
# Pod CPU usage in retail namespace
rate(container_cpu_usage_seconds_total{namespace="retail"}[5m])

# Pod memory usage
container_memory_usage_bytes{namespace="retail"}

# Pod restart count
kube_pod_container_status_restarts_total{namespace="retail"}
```

---

## 🔄 The GitOps Loop

```
┌─────────────────────────────────────────────────────┐
│                  THE GITOPS LOOP                    │
│                                                     │
│  1. Developer: git push                             │
│         ↓                                           │
│  2. GitHub Webhook fires (< 10 seconds)             │
│         ↓                                           │
│  3. Jenkins: Stage 1 — Checkout code                │
│         ↓                                           │
│  4. Jenkins: Stage 2 — Build 5 Docker images        │
│             Tagged with :$BUILD_NUMBER              │
│         ↓                                           │
│  5. Jenkins: Stage 3 — Push images to DockerHub     │
│         ↓                                           │
│  6. Jenkins: Stage 4 — Update values.yaml           │
│             tag: "v1" → tag: "$BUILD_NUMBER"        │
│             git push to retail-store-gitops         │
│         ↓                                           │
│  7. ArgoCD: Detects Git change (within 3 min)       │
│         ↓                                           │
│  8. ArgoCD: Rolling update in KIND cluster          │
│         ↓                                           │
│  9. readinessProbe passes → pod is Ready            │
│         ↓                                           │
│  10. New version LIVE at http://EC2_IP ✅            │
│                                                     │
│  Total time: ~5 minutes                             │
│  Manual intervention: ZERO                         │
└─────────────────────────────────────────────────────┘
```

---

## 💰 Cost Management

**Stop EC2 when not in use:**
```bash
aws ec2 stop-instances --instance-ids YOUR_INSTANCE_ID --region ap-south-1
```

**Start EC2:**
```bash
aws ec2 start-instances --instance-ids YOUR_INSTANCE_ID --region ap-south-1

# Get new IP (wait 30 seconds after starting)
aws ec2 describe-instances \
  --instance-ids YOUR_INSTANCE_ID \
  --region ap-south-1 \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text
```

> ⚠️ **After restart:** EC2 IP changes. Update `terraform.tfvars` and run `terraform apply`.
> Update `inventory.ini` and update Jenkins webhook URL.

**Estimated costs:**
| Resource | Cost |
|---|---|
| t3.large (running) | ~$0.08/hour |
| t3.large (stopped) | ~$0.10/month (EBS only) |
| S3 (state file) | < $0.01/month |

---

## 🎤 Interview Q&A

**Q: Walk me through your project architecture.**
> "It's a 5-service polyglot microservices application — Java Spring Boot for UI, cart, and orders; Go for the catalog service; and Node.js/NestJS for checkout. All services run on a KIND Kubernetes cluster on AWS EC2. Infrastructure is provisioned with Terraform, configured with Ansible, containerised with multi-stage Docker builds, deployed via Helm charts managed by ArgoCD, and built by a Jenkins CI pipeline. A git push triggers the entire pipeline automatically."

**Q: What is the difference between CI and CD?**
> "CI — Continuous Integration — is about automatically building and testing code on every push. Jenkins handles our CI: it builds 5 Docker images and pushes them to DockerHub. CD — Continuous Deployment — is about automatically deploying the built artifact. ArgoCD handles our CD: it watches the GitOps repo and deploys whenever values.yaml changes. They're deliberately separated tools — Jenkins knows about code, ArgoCD knows about Kubernetes."

**Q: Why two repos?**
> "The two-repo GitOps pattern separates application code from deployment config. Repo 1 triggers Jenkins CI on every push. Repo 2 triggers ArgoCD CD when values.yaml changes. This prevents circular triggers, gives clean audit trails for code vs config changes, and allows rollback by simply reverting values.yaml."

**Q: What does selfHeal: true do?**
> "If someone manually scales a deployment with kubectl, ArgoCD detects the drift between Git and the cluster and reverts it automatically. Git is always the source of truth — no manual kubectl changes can persist."

---

## 👨‍💻 Author

**Harjot Singh**
- GitHub: [@HarjotSingh2k19](https://github.com/HarjotSingh2k19)
- LinkedIn: [harjot-singh-579ba9184](https://linkedin.com/in/harjot-singh-579ba9184)
- MCS Student — University of Ottawa (Fall 2026)

---

## 📄 License

MIT License — feel free to use this project as a learning reference.

---

> ⭐ If this project helped you, please give it a star!

