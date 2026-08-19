# ☸️ Kubernetes Deployed Multi-Service Application

A **three-tier containerized web application deployed on Kubernetes using AWS EKS**, consisting of a **React frontend**, **Node.js backend**, and **MongoDB database**.

The project demonstrates how multiple containerized services can communicate inside a Kubernetes cluster using **Deployments, Services, Secrets, health probes, and Ingress**, while application images are stored in **Amazon ECR**.

---

## 📌 Project Overview

This project deploys a complete three-tier application on **Amazon Elastic Kubernetes Service (EKS)**.

The architecture consists of:

* **Frontend** — React application
* **Backend** — Node.js API
* **Database** — MongoDB
* **Container Registry** — Amazon ECR
* **Container Orchestration** — Kubernetes
* **Cloud Platform** — AWS EKS
* **External Traffic Routing** — AWS Application Load Balancer
* **Domain** — `todo.shreemant.im`

Kubernetes manages the application containers, internal networking, service discovery, deployment updates, secrets, and external traffic routing.

---

# 🏗️ Architecture

```text
                         Internet
                            │
                            ▼
                    todo.shreemant.im
                            │
                            ▼
                 AWS Application Load
                       Balancer
                            │
                  Kubernetes Ingress
                            │
             ┌──────────────┴──────────────┐
             │                             │
             │ /                           │ /api
             ▼                             ▼
      Frontend Service               Backend Service
        Port 3000                       Port 3500
             │                             │
             ▼                             ▼
       React Frontend                 Node.js API
                                           │
                                           ▼
                                  MongoDB Service
                                      Port 27017
                                           │
                                           ▼
                                        MongoDB
```

---

# 🔄 Request Flow

```text
User
 │
 ▼
todo.shreemant.im
 │
 ▼
AWS Application Load Balancer
 │
 ▼
Kubernetes Ingress
 │
 ├── /       → Frontend Service → React Pod
 │
 └── /api    → Backend Service  → Node.js API Pod
                                        │
                                        ▼
                                MongoDB Service
                                        │
                                        ▼
                                   MongoDB Pod
```

The Ingress controller routes requests based on the URL path:

```text
/       → frontend:3000
/api    → api:3500
```

---

# 🛠️ Tech Stack

| Technology                       | Purpose                        |
| -------------------------------- | ------------------------------ |
| **React.js**                     | Frontend application           |
| **Node.js**                      | Backend API                    |
| **MongoDB**                      | Database                       |
| **Docker**                       | Application containerization   |
| **Kubernetes**                   | Container orchestration        |
| **AWS EKS**                      | Managed Kubernetes cluster     |
| **Amazon ECR**                   | Docker image registry          |
| **AWS ALB**                      | External load balancing        |
| **AWS Load Balancer Controller** | Kubernetes Ingress integration |
| **kubectl**                      | Kubernetes cluster management  |
| **eksctl**                       | EKS cluster provisioning       |
| **AWS CLI**                      | AWS resource management        |
| **Git & GitHub**                 | Source code management         |

---

# 📂 Project Structure

```text
Kubernetes-deployed-multi-service-app/
│
├── Application-Code/
│   │
│   ├── backend/
│   │
│   └── frontend/
│
├── Kubernetes-Manifests-file/
│   │
│   ├── Backend/
│   │   ├── backend-deployment.yaml
│   │   └── backend-service.yaml
│   │
│   ├── Database/
│   │   ├── deployment.yaml
│   │   ├── secrets.yaml
│   │   └── service.yaml
│   │
│   ├── Frontend/
│   │   ├── frontend-deployment.yaml
│   │   └── frontend-service.yaml
│   │
│   └── ingress.yaml
│
├── assets/
│
└── README.md
```

---

# ☸️ Kubernetes Architecture

All application components are deployed inside the Kubernetes namespace:

```text
workshop
```

The cluster contains three primary workloads:

```text
Frontend
Backend
MongoDB
```

Each application component is managed through Kubernetes resources.

---

# 🎨 Frontend

The frontend is a **React application** running inside a Kubernetes Deployment.

### Deployment

```text
Deployment: frontend
Namespace: workshop
Replicas: 1
Container Port: 3000
```

The application image is stored inside **Amazon ECR**.

```text
three-tier-frontend:latest
```

The frontend communicates with the backend through:

```text
http://todo.shreemant.im/api/tasks
```

The backend endpoint is configured through:

```text
REACT_APP_BACKEND_URL
```

---

## Frontend Service

The frontend is exposed internally through a Kubernetes `ClusterIP` Service.

```text
Service Name: frontend
Type: ClusterIP
Port: 3000
```

The service provides stable internal networking for frontend pods.

---

# ⚙️ Backend API

The backend is deployed as a Node.js API.

### Deployment

```text
Deployment: api
Namespace: workshop
Replicas: 1
Container Port: 3500
```

The Docker image is stored in **Amazon ECR**:

```text
three-tier-backend:latest
```

The deployment uses:

```yaml
imagePullPolicy: Always
```

which ensures Kubernetes checks for the latest container image whenever the pod starts.

---

# 🗄️ Backend → MongoDB Connection

The backend connects to MongoDB through Kubernetes service discovery.

```text
mongodb://mongodb-svc:27017/todo
```

Instead of using an IP address, the application communicates using the Kubernetes Service name:

```text
mongodb-svc
```

This allows Kubernetes DNS to automatically resolve the MongoDB service.

---

# 🔐 Kubernetes Secrets

MongoDB credentials are provided to the application through Kubernetes Secrets.

The backend retrieves:

```text
MONGO_USERNAME
MONGO_PASSWORD
```

from:

```text
mongo-sec
```

Example:

```yaml
valueFrom:
  secretKeyRef:
    name: mongo-sec
    key: username
```

This keeps credentials separate from the application container image.

> In production environments, sensitive secret files should not be committed directly to public Git repositories. Use Kubernetes Secrets with external secret management solutions such as AWS Secrets Manager.

---

# ❤️ Kubernetes Health Checks

The backend deployment includes both **Liveness** and **Readiness Probes**.

The application exposes:

```text
/ok
```

on:

```text
Port 3500
```

---

## Liveness Probe

The liveness probe verifies that the backend container is still functioning.

```text
GET /ok
Port 3500
```

If the application becomes unhealthy, Kubernetes can restart the container.

---

## Readiness Probe

The readiness probe determines whether the application is ready to receive traffic.

```text
GET /ok
Port 3500
```

Traffic is only sent to pods that pass the readiness check.

---

# 🔁 Rolling Updates

Both the frontend and backend deployments use:

```yaml
strategy:
  type: RollingUpdate
```

The configuration allows Kubernetes to gradually replace existing pods when a new application version is deployed.

Example:

```text
Old Pod
   │
   ├── New Pod Created
   │
   ├── Health Check Passes
   │
   └── Old Pod Removed
```

This reduces downtime during application updates.

---

# 🗃️ MongoDB

MongoDB runs inside the Kubernetes cluster.

### Deployment

```text
Deployment: mongodb
Namespace: workshop
Replicas: 1
Container Port: 27017
```

MongoDB image:

```text
mongo:4.4.6
```

MongoDB credentials are provided through:

```text
mongo-sec
```

using the environment variables:

```text
MONGO_INITDB_ROOT_USERNAME
MONGO_INITDB_ROOT_PASSWORD
```

---

# 🔌 MongoDB Service

MongoDB is exposed internally using a Kubernetes Service.

```text
Service Name: mongodb-svc
Port: 27017
Target Port: 27017
```

The backend can therefore communicate with MongoDB using:

```text
mongodb-svc:27017
```

rather than directly connecting to a pod IP.

---

# 🌐 Kubernetes Services

The project uses `ClusterIP` Services for internal communication.

| Service       |    Port | Purpose          |
| ------------- | ------: | ---------------- |
| `frontend`    |  `3000` | React frontend   |
| `api`         |  `3500` | Node.js backend  |
| `mongodb-svc` | `27017` | MongoDB database |

The frontend and backend are not directly exposed to the internet.

External access is handled through the Kubernetes Ingress.

---

# 🌍 AWS Application Load Balancer

External traffic is routed through an **AWS Application Load Balancer** created using the AWS Load Balancer Controller.

Ingress configuration:

```yaml
ingressClassName: alb
```

The ALB is configured as:

```text
internet-facing
```

and uses:

```text
Target Type: IP
Listener: HTTP 80
```

---

# 🔀 Ingress Routing

The project uses host-based and path-based routing.

Domain:

```text
todo.shreemant.im
```

Routing configuration:

```text
todo.shreemant.im/
        │
        ▼
frontend:3000
```

and:

```text
todo.shreemant.im/api
        │
        ▼
api:3500
```

This allows both frontend and backend traffic to use the same domain.

---

# 🐳 Amazon ECR

The frontend and backend Docker images are stored in **Amazon Elastic Container Registry**.

Example repositories:

```text
three-tier-frontend
three-tier-backend
```

The Kubernetes deployments pull these images directly from ECR.

The manifests use:

```text
ecr-registry-secret
```

as an `imagePullSecret`.

---

# 🚀 Deployment Guide

## 1️⃣ Clone the Repository

```bash
git clone https://github.com/Shreemant-Acharya/Kubernetes-deployed-multi-service-app.git
```

Navigate into the project:

```bash
cd Kubernetes-deployed-multi-service-app
```

---

# 2️⃣ Configure AWS CLI

Install and configure AWS CLI.

```bash
aws configure
```

Provide:

```text
AWS Access Key ID
AWS Secret Access Key
Default Region
Output Format
```

---

# 3️⃣ Create the EKS Cluster

Example cluster configuration:

```bash
eksctl create cluster \
  --name three-tier-cluster \
  --region us-west-2 \
  --node-type t2.medium \
  --nodes-min 2 \
  --nodes-max 2
```

Update your Kubernetes configuration:

```bash
aws eks update-kubeconfig \
  --region us-west-2 \
  --name three-tier-cluster
```

Verify nodes:

```bash
kubectl get nodes
```

---

# 4️⃣ Create Kubernetes Namespace

Create the namespace used by the manifests:

```bash
kubectl create namespace workshop
```

Verify:

```bash
kubectl get namespaces
```

---

# 5️⃣ Create ECR Authentication Secret

If required by your cluster configuration, create an ECR registry secret inside the `workshop` namespace:

```bash
kubectl create secret docker-registry ecr-registry-secret \
  --docker-server=<AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com \
  --docker-username=AWS \
  --docker-password="$(aws ecr get-login-password --region <REGION>)" \
  --namespace workshop
```

Verify:

```bash
kubectl get secrets -n workshop
```

---

# 6️⃣ Deploy MongoDB

```bash
kubectl apply -f Kubernetes-Manifests-file/Database/
```

Verify:

```bash
kubectl get pods -n workshop
```

Check the MongoDB service:

```bash
kubectl get svc -n workshop
```

---

# 7️⃣ Deploy Backend

```bash
kubectl apply -f Kubernetes-Manifests-file/Backend/
```

Verify:

```bash
kubectl get deployment api -n workshop
```

Check pods:

```bash
kubectl get pods -n workshop
```

---

# 8️⃣ Deploy Frontend

```bash
kubectl apply -f Kubernetes-Manifests-file/Frontend/
```

Verify:

```bash
kubectl get deployment frontend -n workshop
```

---

# 9️⃣ Verify All Resources

```bash
kubectl get all -n workshop
```

Expected architecture:

```text
frontend pod
api pod
mongodb pod

frontend service
api service
mongodb-svc service
```

---

# 🔟 Install AWS Load Balancer Controller

Add the EKS Helm repository:

```bash
helm repo add eks https://aws.github.io/eks-charts
```

Update repositories:

```bash
helm repo update
```

Install the AWS Load Balancer Controller after configuring the required IAM role and service account:

```bash
helm install aws-load-balancer-controller \
  eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=three-tier-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

Verify:

```bash
kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller
```

---

# 1️⃣1️⃣ Deploy Ingress

Apply the Ingress resource:

```bash
kubectl apply -f Kubernetes-Manifests-file/ingress.yaml
```

Verify:

```bash
kubectl get ingress -n workshop
```

After the AWS Load Balancer Controller provisions an ALB, an AWS load balancer DNS name should appear.

---

# 🌐 Domain Configuration

The application uses:

```text
todo.shreemant.im
```

Create a DNS record pointing the subdomain to the Application Load Balancer.

Traffic then follows:

```text
todo.shreemant.im
        │
        ▼
AWS ALB
        │
        ▼
Kubernetes Ingress
        │
        ├── / → frontend
        │
        └── /api → backend
```

---

# 🔎 Useful Kubernetes Commands

### Check Pods

```bash
kubectl get pods -n workshop
```

### Check Deployments

```bash
kubectl get deployments -n workshop
```

### Check Services

```bash
kubectl get svc -n workshop
```

### Check Ingress

```bash
kubectl get ingress -n workshop
```

### View Pod Details

```bash
kubectl describe pod <pod-name> -n workshop
```

### View Application Logs

```bash
kubectl logs <pod-name> -n workshop
```

### Watch Pods

```bash
kubectl get pods -n workshop -w
```

### Check All Resources

```bash
kubectl get all -n workshop
```

---

# 🔄 Updating the Application

After building and pushing a new image to Amazon ECR:

```text
three-tier-frontend:latest
```

or:

```text
three-tier-backend:latest
```

restart the deployment:

```bash
kubectl rollout restart deployment frontend -n workshop
```

```bash
kubectl rollout restart deployment api -n workshop
```

Monitor the rollout:

```bash
kubectl rollout status deployment/frontend -n workshop
```

```bash
kubectl rollout status deployment/api -n workshop
```

---

# 🧪 Troubleshooting

## Check Pod Status

```bash
kubectl get pods -n workshop
```

---

## Check Pod Logs

```bash
kubectl logs <pod-name> -n workshop
```

---

## Check Backend Health

```bash
kubectl port-forward service/api 3500:3500 -n workshop
```

Then access:

```text
http://localhost:3500/ok
```

---

## Check Frontend Locally

```bash
kubectl port-forward service/frontend 3000:3000 -n workshop
```

Then open:

```text
http://localhost:3000
```

---

## Check MongoDB Service

```bash
kubectl get svc mongodb-svc -n workshop
```

---

## Check Ingress

```bash
kubectl describe ingress mainlb -n workshop
```

---

# 🧹 Cleanup

Delete the Kubernetes workloads:

```bash
kubectl delete -f Kubernetes-Manifests-file/ingress.yaml
```

```bash
kubectl delete -f Kubernetes-Manifests-file/Frontend/
```

```bash
kubectl delete -f Kubernetes-Manifests-file/Backend/
```

```bash
kubectl delete -f Kubernetes-Manifests-file/Database/
```

Delete the namespace:

```bash
kubectl delete namespace workshop
```

---

# ☁️ Delete EKS Cluster

To avoid unnecessary AWS charges:

```bash
eksctl delete cluster \
  --name three-tier-cluster \
  --region us-west-2
```

Also verify that associated AWS resources such as:

```text
Application Load Balancers
Target Groups
Security Groups
ECR repositories
EC2 resources
```

are no longer required.

---

# 🎯 DevOps Concepts Demonstrated

This project demonstrates practical experience with:

* Kubernetes Deployments
* Kubernetes Services
* Kubernetes Secrets
* Kubernetes Namespaces
* Container networking
* Kubernetes DNS/service discovery
* Liveness probes
* Readiness probes
* Rolling deployments
* Multi-container application architecture
* Docker containerization
* Amazon ECR
* Amazon EKS
* AWS Load Balancer Controller
* Application Load Balancer
* Kubernetes Ingress
* AWS infrastructure

---

# 🔮 Future Improvements

Possible improvements include:

* Horizontal Pod Autoscaling
* Helm chart packaging
* HTTPS using AWS Certificate Manager
* Terraform infrastructure provisioning
* GitHub Actions CI/CD
* Persistent Volumes for MongoDB
* Amazon DocumentDB or MongoDB Atlas
* Prometheus and Grafana monitoring

---

# 👨‍💻 Author

**Shreemant Acharya**

GitHub: `Shreemant-Acharya`

---

# ⭐ Project Summary

> Deployed a three-tier containerized application consisting of React, Node.js, and MongoDB on Amazon EKS using Kubernetes Deployments, Services, Secrets, health probes, and ALB Ingress. Container images are stored in Amazon ECR, internal communication is handled through Kubernetes service discovery, and external traffic is routed through an AWS Application Load Balancer using `todo.shreemant.im`.

