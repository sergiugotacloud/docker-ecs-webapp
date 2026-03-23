# Dockerized Web Application on AWS ECS Fargate
### Docker · ECS · Fargate · Application Load Balancer

[![AWS](https://img.shields.io/badge/AWS-Container-orange?logo=amazon-aws)](https://aws.amazon.com/)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker)](https://www.docker.com/)
[![ECS](https://img.shields.io/badge/Amazon-ECS-FF9900?logo=amazon-aws)](https://aws.amazon.com/ecs/)
[![Fargate](https://img.shields.io/badge/AWS-Fargate-yellow?logo=amazon-aws)](https://aws.amazon.com/fargate/)
[![ALB](https://img.shields.io/badge/Amazon-Load%20Balancer-8C4FFF?logo=amazon-aws)](https://aws.amazon.com/elasticloadbalancing/)

---

## Overview

This project demonstrates how to containerize a web application using Docker and deploy it to AWS using ECS Fargate behind an Application Load Balancer.

A static web application is packaged into a Docker image, pushed to Docker Hub, and deployed as a Fargate task inside an ECS service — with no servers to manage, no runtime infrastructure to patch, and full elastic scaling built in.

The goal was to simulate a production-style container deployment while gaining hands-on experience with containerized workloads and AWS container services.

---

## Architecture

![Production Architecture](screenshots/00-production-architecture.png)

```
User (Browser)
      │
      ▼
Application Load Balancer  ── Handles inbound HTTP traffic
      │
      ▼
Target Group  ── Routes requests, performs health checks
      │
      ▼
ECS Service  ── Manages task count and placement
      │
      ▼
Fargate Task  ── Serverless container execution
      │
      ▼
Docker Container (NGINX)  ── Serves static web application
      │
      ▼
Web Application (index.html)
```

The container image is stored in Docker Hub and pulled by ECS during deployment.

---

## Services Used

| Service | Role |
|---|---|
| **Docker** | Builds and packages the web application into a container image |
| **Docker Hub** | Stores and serves the container image for ECS to pull |
| **Amazon ECS** | Orchestrates container deployment and service management |
| **AWS Fargate** | Provides serverless compute for running ECS tasks |
| **Application Load Balancer** | Distributes traffic to healthy Fargate tasks |
| **Target Groups** | Routes ALB traffic and performs container health checks |
| **Security Groups** | Controls inbound and outbound network access |

---

## Application Flow

1. Docker image is built locally and tested as a running container
2. Image is pushed to Docker Hub as the container registry
3. ECS cluster is created as the deployment environment
4. Task Definition specifies the container image, ports, and resource requirements
5. ECS Service is deployed using Fargate as the launch type
6. Application Load Balancer is attached and configured with a Target Group
7. ALB health checks confirm tasks are healthy and serving traffic
8. Application is accessible publicly via the ALB DNS name

---

## Project Structure

```
ecs-fargate-web-app/
│
├── README.md
│
└── screenshots/
    ├── 00-production-architecture.png   # End-to-end architecture diagram
    ├── 01-project-structure.png         # Local project layout
    ├── 02-docker-image-built.png        # Docker build output
    ├── 03-container-running.png         # Local container run confirmation
    ├── 04-docker-webapp-running.png     # Application running locally in browser
    ├── 05-dockerhub-repository.png      # Docker Hub image repository
    ├── 06-ecs-cluster-created.png       # ECS cluster configuration
    ├── 07-task-definition-created.png   # ECS Task Definition settings
    ├── 08-ecs-service-running.png       # ECS Service with running tasks
    ├── 09-alb-overview.png             # Application Load Balancer overview
    ├── 10-target-group-healthy.png      # Target Group with healthy targets
    └── 11-ecs-app-running.png           # Live application via ALB
```

---

## Screenshots

### 1. Project Structure
*Local project files including Dockerfile and application source before containerisation.*

![Project Structure](screenshots/01-project-structure.png)

---

### 2. Docker Image Built
*Docker image successfully built locally from the project Dockerfile.*

![Docker Image](screenshots/02-docker-image-built.png)

---

### 3. Container Running
*Container confirmed running locally after `docker run`, exposing the application on the expected port.*

![Container Running](screenshots/03-container-running.png)

---

### 4. Local Web Application
*Web application loading correctly in the browser from the locally running Docker container.*

![Docker Web App](screenshots/04-docker-webapp-running.png)

---

### 5. Docker Hub Repository
*Container image pushed to Docker Hub and visible in the repository, ready for ECS to pull.*

![Docker Hub](screenshots/05-dockerhub-repository.png)

---

### 6. ECS Cluster Created
*ECS cluster provisioned in AWS as the deployment target for the containerised application.*

![ECS Cluster](screenshots/06-ecs-cluster-created.png)

---

### 7. Task Definition
*Task Definition configured with the Docker Hub image reference, container port mapping, and Fargate launch type.*

![Task Definition](screenshots/07-task-definition-created.png)

---

### 8. ECS Service Running
*ECS Service deployed with the desired task count running and reporting healthy status.*

![ECS Service](screenshots/08-ecs-service-running.png)

---

### 9. Application Load Balancer
*ALB provisioned and active, configured to forward traffic to the ECS Target Group.*

![ALB](screenshots/09-alb-overview.png)

---

### 10. Target Group Healthy
*All registered Fargate targets passing health checks and showing as healthy in the Target Group.*

![Target Group](screenshots/10-target-group-healthy.png)

---

### 11. Application Running via ALB
*Web application successfully accessible via the ALB DNS name — end-to-end deployment confirmed.*

![Application](screenshots/11-ecs-app-running.png)

---

## Troubleshooting

### 1. Container Port Mismatch Causing Failed Health Checks

After deploying the ECS Service and attaching the Load Balancer, the Target Group was reporting all targets as unhealthy. The ECS tasks were launching successfully, but the ALB health check was timing out on every attempt.

The root cause was a mismatch between the container port defined in the Task Definition and the port the health check was probing. The Task Definition had the container mapped to port 80, but NGINX inside the image was configured to listen on port 8080. The health check was hitting port 80, receiving no response, and marking the target unhealthy — causing the ECS Service to cycle tasks continuously.

**Fix:** Updated the Task Definition container port mapping to match the port NGINX was actually listening on, and aligned the Target Group health check port to the same value. Redeployed the service and confirmed targets moved to healthy within two check intervals.

**Lesson:** Always verify that the container port in the Task Definition, the Target Group health check port, and the port the application is actually bound to inside the container are all consistent. A mismatch at any point in this chain produces the same symptom — unhealthy targets — but for a different reason each time.

---

### 2. Security Group Rules Blocking ALB-to-Container Traffic

With health checks passing, external requests to the ALB DNS name were timing out without reaching the application. The ALB itself was returning no response, and CloudWatch showed no Lambda invocations or error events — the traffic wasn't getting through at all.

The issue was the security group attached to the ECS tasks. It was configured to allow inbound traffic only from specific IP ranges used during local testing, and had not been updated to allow traffic from the ALB's security group. The ALB was routing requests correctly, but the Fargate tasks were silently dropping the packets at the network layer before any application code was reached.

**Fix:** Updated the ECS task security group to add an inbound rule explicitly allowing traffic from the ALB security group on the container port. Removing the IP-based rule and replacing it with a security group reference is the correct pattern — it ensures only the ALB can reach the tasks, and the rule stays valid even when ALB nodes scale or change IPs.

**Lesson:** In VPC-based architectures, always use security group references rather than IP ranges for inter-service traffic rules. Reference the ALB security group as the source in the task security group — this is both more secure and more maintainable than static CIDR rules.

---

## What I Learned

This project reinforced that containerisation and orchestration are distinct disciplines — and that understanding both is necessary to ship reliably.

**Docker is an isolation and portability tool, not a deployment tool.** Building and running a container locally is straightforward, but the choices made during image construction — which port to bind, which base image to use, how environment is injected — cascade through every downstream deployment decision. Getting the Dockerfile right before touching AWS saved significant debugging time later.

**ECS task definitions are more than configuration — they're the contract between your image and the platform.** Port mappings, resource limits, environment variables, and IAM role references all live in the Task Definition. Treating it as a formal specification rather than a checklist item made it easier to reason about why tasks were behaving unexpectedly.

**Load balancers add a layer of indirection that makes debugging harder without the right mental model.** When the application wasn't reachable, the failure could have been at the ALB listener, the target group routing, the security group, or the container itself. Working through each layer systematically — rather than assuming the first thing checked was the problem — was the only reliable approach.

**Security groups in AWS are stateful and directional, and that matters.** Allowing outbound traffic from a task doesn't automatically permit the corresponding inbound response unless the rule is correctly scoped. Understanding the difference between inbound rules on the task security group and inbound rules on the ALB security group, and how they interact, is fundamental to getting container networking right in a VPC.

---

## Future Improvements

- CI/CD pipeline with GitHub Actions to automate image builds and ECS deployments on push
- Infrastructure as Code with Terraform to make the stack reproducible and version-controlled
- CloudWatch Container Insights for task-level metrics and structured log aggregation
- HTTPS with AWS Certificate Manager and an ACM-backed ALB listener
- Auto Scaling policy tied to ALB request count or CPU utilisation

---

## Author

**Sergiu Gota**
AWS Certified Solutions Architect – Associate · AWS Cloud Practitioner

[![GitHub](https://img.shields.io/badge/GitHub-sergiugotacloud-181717?logo=github)](https://github.com/sergiugotacloud)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-sergiu--gota--cloud-0A66C2?logo=linkedin)](https://linkedin.com/in/sergiu-gota-cloud)

> Built as part of a cloud portfolio to demonstrate real-world containerised application deployment on AWS.
> Feel free to fork, adapt, or reach out with questions.
