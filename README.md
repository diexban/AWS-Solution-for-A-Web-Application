# AWS Solution For A Web Application

## Infrastructure Design Proposal for AWS Web Application Hosting

As part of the infrastructure design for the new web application, I am proposing **two solutions** for hosting the application on AWS. Since it is not yet confirmed whether the application will be built using container images, the two solutions address both possibilities:

1. **Using AWS Fargate**  
   _(For container-based workloads)_  

2. **Using EC2 Instances**  
   _(For traditional virtual machines)_

## Solution #1

![AWS Solution Fargate (2)](https://github.com/user-attachments/assets/babedf79-e1fc-4622-93a0-9339c00334ae)

## Explanation and Components Used in the Architecture Diagram

For my first solution, I decided to create a VPC within a region and distribute components across two Availability Zones to ensure high availability and fault tolerance. I envisioned the services being built with a containerization mindset, so I created two ECS clusters: one for the web layer and one for the application layer.

In each Availability Zone, I deployed Fargate instances running both front-end and back-end services using images pulled from Amazon ECR. These instances are separated into their own clusters and subnets to ensure they have the necessary resources and network ranges to scale based on traffic and computing load.

To handle load distribution between the web and app layers, I set up an internal load balancer that balances traffic between the web layer (in ECS) and the app layer. Each Fargate instance is placed inside its own security group, allowing communication only between layers and the necessary services. This separation minimizes the risk of security breaches.

For the database layer, I have set up an RDS instance with synchronous replication in the second Availability Zone, which ensures failover capabilities in case of an issue. This database is also placed inside its own security group for added security. I opted for a single RDS instance, as it has sufficient capacity to scale on its own. Additionally, I added an ElastiCache Redis node to offload some of the load from the database using caching.

For content distribution, I use CloudFront to accelerate delivery and cache static content. CloudFront is connected to an Application Load Balancer in a public subnet of my VPC, which balances traffic to the web layer. For storage, I leverage S3 to serve static content, store backups, and store logs.

In terms of monitoring, I configured CloudWatch to monitor the health of each component and CloudTrail to track any significant changes made to the environment, helping with auditing and troubleshooting.


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
- **AWS CloudTrail** (for auditing )
- **AWS IAM** (Identity and Access Management for security control)

### 8. Deployment & Container Registry
- **Amazon Elastic Container Registry (ECR)** (for container image storage)

##  Discussion


