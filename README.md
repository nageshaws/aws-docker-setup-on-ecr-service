# aws-docker-setup-on-ecr-service:

# AWS Docker Compose Full-Stack Deployment

Deploying a production-ready full-stack application on AWS EC2 using Docker Compose, AWS ECR, MongoDB, Traefik SSL Reverse Proxy, and Portainer.

## Project Overview

This project demonstrates containerized deployment of a full-stack application using Docker technologies and AWS cloud services.

### Components

* Frontend Application
* Backend REST API
* MongoDB Database
* Traefik Reverse Proxy
* Let's Encrypt SSL Certificates
* Portainer Container Management
* AWS EC2
* AWS ECR Container Registry

---

## Architecture

```text
                    Internet
                        │
                        ▼
                Traefik Reverse Proxy
                  (SSL/TLS Enabled)
                        │
          ┌─────────────┴─────────────┐
          │                           │
          ▼                           ▼
     Frontend Container       Backend API Container
                                      │
                                      ▼
                             MongoDB Container

                        Portainer Dashboard
```

---

## Prerequisites

### AWS Resources

* AWS Account
* EC2 Instance (Ubuntu 22.04)
* IAM User with ECR Access
* Domain Name (Optional)
* Elastic IP (Recommended)

### Open Security Group Ports

| Port | Purpose                      |
| ---- | ---------------------------- |
| 22   | SSH                          |
| 80   | HTTP                         |
| 443  | HTTPS                        |
| 9000 | Portainer                    |
| 8080 | Traefik Dashboard (Optional) |

---

# Step 1: Launch EC2 Instance

Connect to EC2:

```bash
ssh -i my-key.pem ubuntu@PUBLIC_IP
```

Update packages:

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Step 2: Install Docker

```bash
curl -fsSL https://get.docker.com | sudo sh
```

Verify:

```bash
docker --version
```

Add current user:

```bash
sudo usermod -aG docker ubuntu
newgrp docker
```

Test:

```bash
docker run hello-world
```

---

# Step 3: Install Docker Compose

```bash
sudo apt install docker-compose-plugin -y
```

Verify:

```bash
docker compose version
```

---

# Step 4: Install AWS CLI

```bash
sudo apt install unzip -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
-o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install
```

Verify:

```bash
aws --version
```

Configure:

```bash
aws configure
```

Enter:

```text
AWS Access Key
AWS Secret Key
Region (ap-south-1)
Output Format (json)
```

---

# Step 5: Create AWS ECR Repository

Frontend Repository:

```bash
aws ecr create-repository \
--repository-name frontend
```

Backend Repository:

```bash
aws ecr create-repository \
--repository-name backend
```

List repositories:

```bash
aws ecr describe-repositories
```

---

# Step 6: Authenticate Docker to ECR

```bash
aws ecr get-login-password \
--region ap-south-1 \
| docker login \
--username AWS \
--password-stdin ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com
```

---

# Step 7: Build Frontend Image

```bash
cd frontend

docker build -t frontend:v1 .
```

Tag:

```bash
docker tag frontend:v1 \
ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/frontend:v1
```

Push:

```bash
docker push \
ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/frontend:v1
```

---

# Step 8: Build Backend Image

```bash
cd backend

docker build -t backend:v1 .
```

Tag:

```bash
docker tag backend:v1 \
ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/backend:v1
```

Push:

```bash
docker push \
ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/backend:v1
```

---

# Step 9: Install Portainer

Create volume:

```bash
docker volume create portainer_data
```

Run Portainer:

```bash
docker run -d \
--name portainer \
-p 9000:9000 \
-p 9443:9443 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data \
portainer/portainer-ce
```

Access:

```text
http://SERVER_IP:9000
```

---

# Step 10: Create Docker Compose File

docker-compose.yml

```yaml
services:

  frontend:
    image: ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/frontend:v1
    restart: always

  backend:
    image: ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/backend:v1
    restart: always

  mongodb:
    image: mongo:7
    restart: always
    volumes:
      - mongodb_data:/data/db

  traefik:
    image: traefik:v3
    restart: always
    ports:
      - "80:80"
      - "443:443"

volumes:
  mongodb_data:
```

---

# Step 11: Deploy Containers

Start:

```bash
docker compose up -d
```

Verify:

```bash
docker compose ps
```

Check running containers:

```bash
docker ps
```

---

# Step 12: Monitor Containers

Logs:

```bash
docker compose logs -f
```

Specific service:

```bash
docker logs backend
```

Container shell:

```bash
docker exec -it backend sh
```

Resource usage:

```bash
docker stats
```

---

# Step 13: Troubleshooting

View stopped containers:

```bash
docker ps -a
```

Restart service:

```bash
docker restart backend
```

Recreate stack:

```bash
docker compose down

docker compose up -d
```

---

# Production Use Cases

### Continuous Deployment

Build new image:

```bash
docker build -t backend:v2 .
```

Push:

```bash
docker push backend:v2
```

Update compose file:

```yaml
image: backend:v2
```

Deploy:

```bash
docker compose up -d
```

---

### Monitoring

Using Portainer:

* View logs
* Restart containers
* Monitor CPU
* Monitor Memory
* Deploy stacks
* Troubleshoot failures

---

### SSL Management

Traefik:

* Automatic HTTPS
* Automatic Certificate Renewal
* HTTP → HTTPS Redirection

---

## Useful Commands

```bash
docker ps

docker images

docker network ls

docker volume ls

docker stats

docker logs -f container_name

docker exec -it container_name sh

docker system prune -a
```

---

## Technologies Used

* Docker
* Docker Compose
* AWS EC2
* AWS ECR
* MongoDB
* Traefik
* Portainer
* Linux
* Networking
* SSL/TLS
* DevOps

---

## Learning Outcomes

* Containerization
* Multi-container deployments
* AWS ECR image management
* Reverse proxy configuration
* SSL automation
* Container monitoring
* Infrastructure troubleshooting
* Production deployment workflows
