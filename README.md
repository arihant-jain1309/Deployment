README: Deploy Portfolio & Snake-Bite Game on AWS EKS

Stage 1 — Setup Ubuntu Environment

sudo apt update && sudo apt install -y curl unzip git docker.io

sudo systemctl enable --now docker
sudo usermod -aG docker $USER

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip"
unzip /tmp/awscliv2.zip -d /tmp
sudo /tmp/aws/install
aws --version
aws configure

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm version

Stage 2 — Build & Push Docker Images

docker login

# Portfolio
docker build -t YOUR_USERNAME/portfolio:latest ./Arihant-Portfolio
docker push YOUR_USERNAME/portfolio:latest

# Snake-Bite Game
docker build -t YOUR_USERNAME/snake-bite-game:latest ./Snake-Bite-Game
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

kubectl get nodes

kubectl cluster-info

Stage 4 — Deploy Applications to Kubernetes

kubectl apply -f portfolio-deployment.yaml

kubectl apply -f snake-deployment.yaml

kubectl get pods -o wide

kubectl get svc

Stage 5 — Expose Apps with Ingress

kubectl create namespace ingress-nginx

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm repo update

helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx

kubectl apply -f apps-ingress.yaml
kubectl get ingress
