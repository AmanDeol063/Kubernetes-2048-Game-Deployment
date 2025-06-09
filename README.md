# Kubernetes 2048 Game Deployment

A hands-on project showcasing the deployment of the classic **2048 game** as a React application on **Kubernetes** (EKS + Fargate). Follow this guide to clone the repository, containerize the app, and deploy it on your own EKS cluster.

---

## 📋 Table of Contents

* [Features](#features)
* [Tech Stack](#tech-stack)
* [Prerequisites](#prerequisites)
* [Getting Started](#getting-started)

  * [1. Clone the Repo](#1-clone-the-repo)
  * [2. Build & Containerize](#2-build--containerize)
  * [3. Push to Container Registry](#3-push-to-container-registry)
  * [4. Deploy to Kubernetes (EKS + Fargate)](#4-deploy-to-kubernetes-eks--fargate)
* [Accessing the App](#accessing-the-app)
* [(Optional) Serverless Scoreboard](#optional-serverless-scoreboard)
* [Cleanup](#cleanup)
* [License](#license)

---

## 🎯 Features

* Interactive **2048** gameplay in React
* Responsive grid layout with smooth animations
* Random tile spawning, sliding, and merging logic
* Kubernetes deployment manifests (Deployment, Service, Ingress)
* AWS ALB Ingress Controller integration for load balancing
* Optional serverless backend (Lambda + DynamoDB) for score persistence

---

## 🧰 Tech Stack

* **Frontend:** React, CSS Grid
* **Containerization:** Docker
* **Kubernetes:** EKS, Fargate
* **Ingress:** AWS ALB Ingress Controller
* **Optional Backend:** AWS Lambda, API Gateway, DynamoDB

---

## ⚙️ Prerequisites

* AWS account with Free Tier enabled
* AWS CLI configured (`aws configure`)
* `kubectl` installed and configured for your EKS cluster
* `eksctl` (optional) for cluster creation
* Docker installed locally
* Git client

---

## 🚀 Getting Started

### 1. Clone the Repo

```bash
git clone https://github.com/<your-username>/kubernetes-2048-game-deployment.git
cd kubernetes-2048-game-deployment
```

### 2. Build & Containerize

1. Build the React app and create a Docker image:

   ```bash
   docker build -t <your-docker-id>/2048-game:latest .
   ```

2. Run locally to verify:

   ```bash
   docker run -p 8080:80 <your-docker-id>/2048-game:latest
   # Open http://localhost:8080
   ```

### 3. Push to Container Registry

1. Log in to Docker Hub (or ECR):

   ```bash
   docker login  # for Docker Hub
   # or
   aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
   ```
2. Tag & push:

   ```bash
   docker push <your-docker-id>/2048-game:latest
   # or for ECR:
   docker tag 2048-game:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/2048-game:latest
   docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/2048-game:latest
   ```

### 4. Deploy to Kubernetes (EKS + Fargate)

1. Ensure your kubeconfig points to the target EKS cluster:

   ```bash
   aws eks update-kubeconfig --region <region> --name <cluster-name>
   ```

2. Apply manifests:

   ```bash
   kubectl apply -f k8s/namespace.yaml
   kubectl apply -f k8s/deployment.yaml
   kubectl apply -f k8s/service.yaml
   kubectl apply -f k8s/ingress.yaml
   ```

3. Verify pods, service, and ingress:

   ```bash
   kubectl get pods -n 2048-game
   kubectl get svc -n 2048-game
   kubectl get ingress -n 2048-game
   ```


## 🧹 Cleanup

To avoid AWS charges:

```bash
# Kubernetes
kubectl delete namespace 2048-game

# Docker image
docker rmi <your-docker-id>/2048-game

# (Optional) Delete ECR repo / IAM roles / DynamoDB / Lambda / API Gateway
