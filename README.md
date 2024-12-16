# AWS Solution For A Web Application

## Infrastructure Design Proposal for AWS Web Application Hosting

As part of the infrastructure design for the new web application, I am proposing **two solutions** for hosting the application on AWS. Since it is not yet confirmed whether the application will be built using container images, the two solutions address both possibilities:

1. **Using AWS Fargate**  
   _(For container-based workloads)_  

2. **Using EC2 Instances**  
   _(For traditional virtual machines)_

## Solution #1

![AWS Solution Fargate (2)](https://github.com/user-attachments/assets/babedf79-e1fc-4622-93a0-9339c00334ae)

## Components Used in the Architecture Diagram

### 1. Networking Layer
- **AWS Region**
- **Virtual Private Cloud (VPC)**
- **Public Subnet** (used for Application Load Balancer)
- **Private Subnets**
- **AWS Availability Zone 1**
- **AWS Availability Zone 2**
- **Security Groups**

### 2. Compute Layer
- **Amazon ECS Cluster 1 in each layer**
- **AWS Fargate** (used in both Web Layer and App Layer)

### 3. Load Balancing
- **Application Load Balancer** (Public-facing)
- **Internal Load Balancer**

### 4. Database Layer
- **Amazon RDS** (Relational Database Service)
- **RDS Synchronous Replica** (for high availability and failover)
- **ElastiCache Redis Nodes** (used as caching layers in both Availability Zones)

### 5. Storage Layer
- **Amazon S3** (for log, backup and static content storage)

### 6. Content Delivery
- **Amazon CloudFront** (used as a CDN for caching and accelerating content)

### 7. Monitoring and Security
- **AWS CloudWatch** (for monitoring and logging)
- **AWS CloudTrail** (for auditing and tracking API calls)
- **AWS IAM** (Identity and Access Management for security control)

### 8. Deployment & Container Registry
- **Amazon Elastic Container Registry (ECR)** (for container image storage)
- **Developer Pipeline** (to automate the creation of the images and the upload to ECR)
