# CI/CD Pipeline with GitHub Actions, Docker & Amazon EKS

## Project Overview

This project implements an automated CI/CD pipeline to build, containerize, and deploy a Spring Boot application to Kubernetes running on Amazon EKS.

### Architecture

```text
GitHub
   ↓
GitHub Actions
   ↓
Self-Hosted Runner (EC2)
   ↓
Maven Build & Test
   ↓
Docker Image
   ↓
Docker Hub
   ↓
Kubernetes / Amazon EKS
   ↓
NodePort
   ↓
Application
```

## 1. GitHub Repository

* Source code is stored in a GitHub repository.
* CI/CD workflow is stored in:
  `.github/workflows/ci-cd.yml`

## 2. Self-Hosted GitHub Runner

* Created a separate Ubuntu EC2 instance.
* Created a user and provided required permissions.
* Added the EC2 instance as a **self-hosted GitHub Actions runner**.
* Executed the commands provided by GitHub to configure the runner.
* Started the runner using:

```bash
./run.sh
```

## 3. CI/CD Pipeline

The GitHub Actions workflow performs:

1. Checkout source code.
2. Build and test using Maven.
3. Build Docker image.
4. Push Docker image to Docker Hub.
5. Copy Kubernetes manifests to the Kubernetes server.
6. Execute `kubectl apply` remotely.

## 4. Amazon EKS

Created an EKS cluster using `eksctl`.

```bash
eksctl create cluster \
  --name jay-cluster \
  --region ap-south-1 \
  --node-type t3.small \
  --nodes 2
```

Verify the cluster:

```bash
kubectl get nodes
```

## 5. Kubernetes Deployment

Kubernetes manifests contain:

* MySQL Deployment
* MySQL Service
* Application Deployment
* NodePort Service

Deploy using:

```bash
kubectl apply -f *.yml
```

Check deployment:

```bash
kubectl get pods
kubectl get svc
```

## 6. GitHub Secrets

The Kubernetes server connection is configured using GitHub Secrets:

```text
K8S_HOST
K8S_USER
K8S_SSH_KEY
DOCKERHUB_TOKEN
```

SSH keys are used by GitHub Actions to securely connect to the Kubernetes server.

## 7. Application Access

The application is exposed using NodePort:

```text
http://<WORKER-NODE-PUBLIC-IP>:<nodeport_number>
```

AWS Security Group must allow TCP traffic on port `nodeport_number`.

## Result

The complete deployment process is automated:

```text
Code Push
   ↓
GitHub Actions
   ↓
Maven Build
   ↓
Docker Build & Push
   ↓
Kubernetes Deployment
   ↓
Amazon EKS
   ↓
Application
```
