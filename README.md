🚀 AWS ECS CI/CD Pipeline Project
📌 Project Overview

This project demonstrates an end-to-end CI/CD pipeline on AWS using GitHub, Docker, Amazon ECR, Amazon ECS (Fargate), Application Load Balancer, AWS CodePipeline, and AWS CodeBuild.

The setup includes:

Manual Docker image build and push

Automated CI/CD pipeline

ECS Fargate service with Auto Scaling

Proper Free Tier cleanup steps

🛠️ EC2 Setup (Initial Docker Image Build)
Launch EC2 Instance

Launch an EC2 instance (Amazon Linux)

Install Git and Docker
yum install git docker -y

Start Docker
service docker start

Clone Application Repository
git clone https://github.com/vikashsuman23/website.git
cd website

Build Docker Image
docker build -t website:latest .

📦 Amazon ECR (Elastic Container Registry)

Go to Amazon ECR

Create repository:

Repository name: website

Create an IAM Role:

Trusted entity: EC2

Permission: AmazonEC2ContainerRegistryFullAccess (or ecr/admin)

Attach this role to the EC2 instance

Select the ECR repository → View push commands

Execute all push commands on EC2

✅ The latest Docker image will be available in the ECR repository.

🚢 Amazon ECS (Elastic Container Service)
Create ECS Cluster

Go to ECS

Create Cluster

Cluster name: websiteclusterv1

Infrastructure: Fargate only

Click Create

Create Task Definition

Create new task definition

Task definition family: website-task-df-v1

Launch type: AWS Fargate

CPU: 0.25 vCPU

Memory: 0.5 GB

Task role: Select appropriate role

Container configuration:

Name: website

Image URI: Select image from ECR

Click Create

Create ECS Service

Go to Cluster → Services

Create service:

Task definition family: website-task-df-v1

Service name: website-service

Launch type: FARGATE (LATEST)

Deployment type: Replica

Desired tasks: 2

Availability zone rebalancing: Unchecked

Networking:

Remove one subnet (e.g., subnet 1c)

Load Balancing:

Enable load balancer

Load balancer name: myecsalb

Target group name: ECSTG

Auto Scaling:

Min tasks: 2

Max tasks: 4

Scaling policy: Target Tracking

Metric: ECSServiceAverageCPUUtilization

Target value: 70

Click Create

🔗 Access the application using the ALB DNS name

🔄 Build Automated CI/CD Pipeline on AWS
Create GitHub Connection

Go to CodePipeline → Settings → Connections

Create connection:

Name: ecsconnection

Provider: GitHub

Install AWS Connector for GitHub

Grant repository access

Save and connect

Create CodeBuild Project

Go to CodeBuild → Create build project

Project name: website

Source provider: GitHub

Repository: website

Webhook:

Event type: PUSH

Buildspec:

Use buildspec.yml

Service role:

Role name: codebuild-website-service-role-23

Grant Admin permissions after creation

Start build and verify success

Create CodePipeline

Go to CodePipeline → Create pipeline

Pipeline name: website

Service role:

AWSCodePipelineServiceRole-ap-south-1-website

Grant Admin permissions

Source Stage

Provider: GitHub

Connection: ecsconnection

Repository: website

Branch: master

Trigger: Push events (filter branch: master)

Build Stage

Provider: AWS CodeBuild

Project name: website

Deploy Stage

Provider: Amazon ECS

Cluster name: websiteclusterv1

Service name: website-service

Image definition file: imagedefinitions.json

✅ Pipeline automatically builds and deploys on every GitHub push.
