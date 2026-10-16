# 🐳 Docker + AWS | Real-Time Collaborative Code Editor

A real-time collaborative code editor built with React, Monaco Editor, Yjs, Socket.IO, and Node.js, containerized using Docker and integrated with AWS services for container image storage and deployment.

## 📌 Project Overview

This project demonstrates how to containerize a full-stack application using Docker and prepare it for deployment on AWS. It combines a collaborative code editor with a container-based deployment workflow using Amazon Elastic Container Registry (ECR) and Amazon ECS with AWS Fargate.

The application uses React and Monaco Editor for the frontend, Yjs for collaborative document synchronization, Socket.IO for real-time communication, and Node.js with Express for the backend.

## ✨ Features

* 💻 Browser-based code editing using Monaco Editor.
* 🔄 Real-time collaborative editing using Yjs and Socket.IO.
* ⚙️ Node.js and Express backend.
* 🐳 Docker-based application containerization.
* 📦 Multi-stage Docker builds to optimize the final image.
* 🏗️ Docker Buildx for platform-specific image builds.
* ☁️ Amazon ECR integration for storing Docker images.
* 🚀 AWS ECS and Fargate deployment workflow.
* ⚖️ Application Load Balancer integration for public traffic routing when configured.

## 🛠️ Tech Stack

| Technology                    | Purpose                                |
| ----------------------------- | -------------------------------------- |
| React                         | Frontend development                   |
| Monaco Editor                 | Code editing interface                 |
| Yjs                           | Collaborative document synchronization |
| Socket.IO                     | Real-time communication                |
| Node.js                       | Backend runtime                        |
| Express.js                    | HTTP server framework                  |
| Docker                        | Application containerization           |
| Docker Buildx                 | Platform-specific image builds         |
| Amazon ECR                    | Container image registry               |
| Amazon ECS                    | Container orchestration                |
| AWS Fargate                   | Serverless container compute           |
| AWS Application Load Balancer | Traffic distribution when configured   |

## 🏗️ Architecture

```text
                 👤 User / Browser
                        |
                        v
               ┌───────────────────┐
               │   React Frontend  │
               │   Monaco Editor   │
               └───────────────────┘
                        |
                        v
               ┌───────────────────┐
               │  Yjs + Socket.IO  │
               │   Real-Time Sync  │
               └───────────────────┘
                        |
                        v
               ┌───────────────────┐
               │ Node.js + Express │
               │      Backend      │
               └───────────────────┘
                        |
                        v
               ┌───────────────────┐
               │    Docker Image   │
               └───────────────────┘
                        |
                        v
               ┌───────────────────┐
               │    Amazon ECR     │
               └───────────────────┘
                        |
                        v
               ┌───────────────────┐
               │ Amazon ECS/Fargate│
               └───────────────────┘
                        |
                        v
               ┌───────────────────┐
               │ Application Load  │
               │ Balancer (Optional)│
               └───────────────────┘
```

> **Note:** This diagram represents the intended deployment architecture. Actual AWS resources and public accessibility depend on the deployment configuration.

## 📂 Project Structure

```text
docker-aws-main/
├── Backend/
│   ├── public/
│   ├── package.json
│   ├── package-lock.json
│   └── server.js
├── Frontend/
│   └── ...
├── dockerfile
├── .gitignore
└── README.md
```

## ⚙️ Prerequisites

Install the following tools before starting:

* Git
* Node.js
* Docker Desktop
* AWS CLI
* An AWS account with appropriate permissions for Amazon ECR and, if deploying, Amazon ECS.

### Verify the Installations

```bash
git --version
node --version
npm --version
docker --version
docker buildx version
aws --version
```

Ensure Docker Desktop is running before executing Docker commands.

## 🐳 Docker Setup

### 1. Build the Docker Image

Run the command from the project root directory containing the Dockerfile:

```bash
docker build -t docker-aws/server:latest .
```

### 2. Build for Linux AMD64

To build an image for Linux AMD64 architecture:

```bash
docker buildx build --platform linux/amd64 -t docker-aws/server:latest --load .
```

Verify the image architecture:

```bash
docker image inspect docker-aws/server:latest --format '{{.Os}}/{{.Architecture}}'
```

Expected output:

```text
linux/amd64
```

### 3. Run the Container Locally

If the application listens on port `3000` inside the container:

```bash
docker run -d -p 4000:3000 --name collaborative-editor docker-aws/server:latest
```

Open the application in your browser:

```text
http://localhost:4000
```

### 4. Verify the Container

**Check running containers:**

```bash
docker ps
```

**View container logs:**

```bash
docker logs collaborative-editor
```

**Stop the container:**

```bash
docker stop collaborative-editor
```

**Remove the stopped container:**

```bash
docker rm collaborative-editor
```

## ☁️ Amazon ECR Integration

Amazon Elastic Container Registry (ECR) is a managed AWS service used to store and distribute container images.

### 1. Configure AWS CLI

```bash
aws configure
```

Configure credentials for an IAM identity with the required permissions.

### 2. Create an ECR Repository

Create the repository if it does not already exist:

```bash
aws ecr create-repository --repository-name docker-aws/server --region us-east-1
```

If the repository already exists, use the existing repository instead of creating a duplicate.

### 3. Authenticate Docker to ECR

Replace `<AWS_ACCOUNT_ID>` with your AWS account ID:

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

### 4. Tag the Docker Image

```bash
docker tag docker-aws/server:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/docker-aws/server:latest
```

### 5. Push the Image to ECR

```bash
docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/docker-aws/server:latest
```

### 6. Verify the Uploaded Image

```bash
aws ecr describe-images --repository-name docker-aws/server --region us-east-1 --query 'imageDetails[*].[imageTags,imageDigest,imagePushedAt]' --output table
```

You can also verify the image through:

**AWS Console → Amazon ECR → Private repositories → docker-aws/server → Images**

### Understanding Docker Image Layers

During an image push, Docker may display:

* **Layer already exists** — The layer is already available in ECR.
* **Pushed** — A layer was uploaded successfully.
* **latest: digest: sha256:...** — The registry returned the image manifest digest.

Existing layers do not necessarily mean that an old image was pushed. Docker reuses layers whenever possible. Verify the image digest, tag, and architecture when troubleshooting image versions.

## 🚀 AWS ECS + Fargate Deployment

Amazon ECS manages containerized applications, while AWS Fargate provides serverless container compute without requiring you to manage the underlying servers.

### Deployment Workflow

1. Build the Docker image locally.
2. Push the image to Amazon ECR.
3. Create an Amazon ECS cluster configured for Fargate.
4. Create a task definition referencing the ECR image.
5. Configure CPU, memory, container ports, environment variables, and IAM task roles.
6. Create an ECS service to run the application.
7. Configure the VPC, subnets, and security groups.
8. Configure an Application Load Balancer and target group if public access is required.
9. Verify ECS task health, application logs, target group health, and application accessibility.

> **Deployment Status:** These are deployment instructions, not confirmation that the ECS service, Fargate tasks, or load balancer have already been deployed.

## 🔐 Security Best Practices

* Use IAM identities with least-privilege permissions.
* Never commit AWS access keys, credentials, or secrets to GitHub.
* Exclude `.env` files containing sensitive information.
* Restrict inbound traffic through security groups.
* Use HTTPS for public production applications.
* Store production secrets in an appropriate secrets-management service.
* Use specific image tags or digests for reproducible deployments.
* Monitor AWS resource usage and billing.

## 🧠 Key Learnings

* Docker image creation and container execution.
* Multi-stage Docker builds.
* Docker image tagging and architecture selection.
* Docker Buildx for platform-specific builds.
* Amazon ECR authentication and image publishing.
* AWS CLI usage for container registry operations.
* ECS task definitions and Fargate deployment concepts.
* Container networking and load balancer configuration.
* Troubleshooting Docker and AWS deployment issues.

## 🙏 Acknowledgements

This project was developed as a hands-on learning exercise inspired by the following tutorial from Sheryians Coding School:

🎥 **Complete Docker + AWS | Deploy Your First Project in One Shot (Beginners)**

Thanks to the tutorial creators for providing educational guidance on Docker and AWS deployment workflows.

## 👨‍💻 Author

**Darshan Agrawal**

* GitHub: [@DarshanAg4217](https://github.com/DarshanAg4217)
* LinkedIn: [Darshan Agrawal](www.linkedin.com/in/darshan-agrawal-43a338260)
