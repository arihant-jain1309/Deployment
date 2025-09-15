# 🚀 AWS EKS Deployment Guide  
**Deploy Portfolio & Snake-Bite Game Applications on Amazon EKS (Kubernetes)**

---

## 📦 Overview
This guide walks you through deploying two applications — **Arihant Portfolio** and **Snake-Bite Game** — on a robust, scalable AWS EKS (Elastic Kubernetes Service) cluster using Docker and Kubernetes best practices.

---

## 🛠️ Prerequisites
- AWS Account
- IAM user with EKS, EC2, S3, and IAM permissions
- Ubuntu server (local or EC2 recommended)

---

## 1️⃣ Setup Ubuntu Environment

```bash
# Update & install dependencies
sudo apt update && sudo apt install -y curl unzip git docker.io

# Enable Docker
sudo systemctl enable --now docker
sudo usermod -aG docker $USER

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip"
unzip /tmp/awscliv2.zip -d /tmp
sudo /tmp/aws/install
aws --version
aws configure  # Enter your credentials

# Install kubectl (Kubernetes CLI)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

# Install eksctl (EKS CLI)
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

# Install Helm (Kubernetes package manager)
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm version
```

---

## 2️⃣ Build & Push Docker Images

```bash
docker login  # Authenticate with Docker Hub

# Build & push Portfolio app
docker build -t YOUR_DOCKER_USERNAME/portfolio:latest ./Arihant-Portfolio
docker push YOUR_DOCKER_USERNAME/portfolio:latest

# Build & push Snake-Bite Game app
docker build -t YOUR_DOCKER_USERNAME/snake-bite-game:latest ./Snake-Bite-Game
docker push YOUR_DOCKER_USERNAME/snake-bite-game:latest
```

---

## 3️⃣ Create EKS Cluster

```bash
export AWS_REGION=ap-south-1

eksctl create cluster \
  --name my-k8s-cluster \
  --region $AWS_REGION \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 3 \
  --managed

kubectl get nodes         # Verify cluster nodes
kubectl cluster-info      # Cluster details
```

---

## 4️⃣ Deploy Applications to Kubernetes

```bash
# Deploy Portfolio
kubectl apply -f portfolio-deployment.yaml

# Deploy Snake-Bite Game
kubectl apply -f snake-deployment.yaml

kubectl get pods -o wide  # Check pod status
kubectl get svc           # Check services
```

---

## 5️⃣ Expose Apps with Ingress Controller

```bash
# Create ingress namespace
kubectl create namespace ingress-nginx

# Add & update Helm repo
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install ingress controller via Helm
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx

# Apply ingress resource
kubectl apply -f apps-ingress.yaml
kubectl get ingress  # View ingress details
```

---

## 🌐 Access Your Applications
- After deploying, retrieve the **external IP of the Ingress controller** and access your applications via the configured hostnames/domains.

---

## 📚 References
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)
- [Kubernetes Official Docs](https://kubernetes.io/docs/home/)
- [Helm Charts](https://helm.sh/docs/)

---

**Feel free to reach out for further assistance or contribute to this repository!**
