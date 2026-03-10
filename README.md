# Dockerized Web Application on AWS ECS Fargate

This project demonstrates how to containerize a web application using Docker and deploy it to AWS using ECS Fargate behind an Application Load Balancer.

The goal of this project was to simulate a production-style container deployment while gaining hands-on experience with containerized workloads and AWS container services.

---

# Production Architecture

![Production Architecture](screenshots/00-production-architecture.png)

Architecture flow:

User Browser  
↓  
Application Load Balancer  
↓  
Target Group  
↓  
ECS Service  
↓  
Fargate Task  
↓  
Docker Container (NGINX)  
↓  
Web Application (index.html)

The container image is stored in Docker Hub and pulled by ECS during deployment.

---

# Technologies Used

- Docker
- Docker Hub
- AWS ECS
- AWS Fargate
- Application Load Balancer
- Target Groups
- Security Groups
- Cloud Networking

---

# Project Workflow

The deployment followed these main steps:

1. Build the Docker image locally
2. Run the container locally for testing
3. Push the image to Docker Hub
4. Create an ECS Cluster
5. Define the ECS Task Definition
6. Deploy an ECS Service using Fargate
7. Attach an Application Load Balancer
8. Configure Target Groups and health checks
9. Verify the running application

---

# Project Screenshots

## Project Structure

![Project Structure](screenshots/01-project-structure.png)

---

## Docker Image Built

![Docker Image](screenshots/02-docker-image-built.png)

---

## Container Running

![Container Running](screenshots/03-container-running.png)

---

## Local Web Application

![Docker Web App](screenshots/04-docker-webapp-running.png)

---

## Docker Hub Repository

![Docker Hub](screenshots/05-dockerhub-repository.png)

---

## ECS Cluster Created

![ECS Cluster](screenshots/06-ecs-cluster-created.png)

---

## Task Definition

![Task Definition](screenshots/07-task-definition-created.png)

---

## ECS Service Running

![ECS Service](screenshots/08-ecs-service-running.png)

---

## Application Load Balancer

![ALB](screenshots/09-alb-overview.png)

---

## Target Group Healthy

![Target Group](screenshots/10-target-group-healthy.png)

---

## Application Running

![Application](screenshots/11-ecs-app-running.png)

---

# Troubleshooting

During deployment several common issues needed troubleshooting:

- Container port configuration
- Security group rules
- Load balancer health checks
- ECS networking configuration
- Docker image accessibility

These troubleshooting steps reflect common real-world container deployment scenarios.

---

# What This Project Demonstrates

This project demonstrates practical experience with:

- Containerizing applications using Docker
- Publishing container images to Docker Hub
- Running containers using AWS ECS
- Using AWS Fargate for serverless container execution
- Exposing services with an Application Load Balancer
- Configuring health checks and target groups
- Troubleshooting networking and deployment issues

---

# Future Improvements

Possible improvements include:

- CI/CD pipeline with GitHub Actions
- Infrastructure as Code with Terraform
- CloudWatch monitoring and logging
- HTTPS with AWS Certificate Manager
- Auto Scaling configuration

---

# Author

Sergiu  
AWS Certified Solutions Architect – Associate  
AWS Certified Cloud Practitioner
