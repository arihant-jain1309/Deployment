Stage 1 — Set up Ubuntu Environment

sudo apt update && sudo apt install -y curl unzip git docker.io

# Start Docker and enable at boot
sudo systemctl enable --now docker
sudo usermod -aG docker $USER  # log out and back in

# Install AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip"
unzip /tmp/awscliv2.zip -d /tmp
sudo /tmp/aws/install
aws --version

# Configure AWS credentials
aws configure

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

# (Optional) Install Helm
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm version



Stage 2 — Build & Push Docker Images
Folder Structure
~/projects/
├── Arihant-Portfolio/
│   ├── index.html
│   ├── css/
│   ├── js/
│   └── Dockerfile
├── Snake-Bite-Game/
│   ├── index.html
│   ├── css/
│   ├── js/
│   └── Dockerfile
└── kubernetes/
    ├── portfolio-deployment.yaml
    └── snake-deployment.yaml



Build & Push

docker login  # Docker Hub
# Portfolio
cd ~/projects/Arihant-Portfolio
docker build -t YOUR_USERNAME/portfolio:latest .
docker push YOUR_USERNAME/portfolio:latest

# Snake Game
cd ~/projects/Snake-Bite-Game
docker build -t YOUR_USERNAME/snake-bite-game:latest .
docker push YOUR_USERNAME/snake-bite-game:latest



Stage 3 — Create EKS Cluster


AWS_REGION=ap-south-1
eksctl create cluster \
  --name my-k8s-cluster \
  --region $AWS_REGION \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 3 \
  --managed


Verify cluster:

Stage 4 — Deploy Applications to Kubernetes

Portfolio Deployment & Service (portfolio-deployment.yaml)
Snake-Bite Game Deployment & Service (snake-deployment.yaml)

Apply deployments:
kubectl apply -f portfolio-deployment.yaml
kubectl apply -f snake-deployment.yaml
kubectl get pods -o wide
kubectl get svc


Stage 5 — Expose Apps with Ingress

Install NGINX Ingress Controller
kubectl create namespace ingress-nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx


Create Ingress(apps-ingress.yaml)

Apply ingress:
kubectl apply -f apps-ingress.yaml
kubectl get ingress


