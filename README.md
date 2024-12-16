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

■ How does each choice contribute to cost-effectiveness against high
availability?
- A key point in my first solution that contributes to cost-effectiveness is the use of Fargate. As a managed service, Fargate automatically scales based on demand, ensuring that the system performs well under varying loads without the need to over-provision resources. This approach is more cost-effective than using EC2 instances, where over-provisioning could lead to unnecessary expenses.

- Another important decision I made was to use RDS with synchronous replication. While having a replica in a second availability zone is cheaper than using two separate RDS instances for high availability, opting for a single RDS instance without a replica could save costs further. However, this would compromise high availability and fault tolerance. The additional cost of a replica I believe is justified to reduce the risk of downtime, which could lead to greater losses.

- Lastly while using CloudFront adds some cost, it provides significant performance benefits by reducing strain on the web layer. With caching in place, the web layer does not need to scale as frequently, keeping costs lower overall. Similarly, using Redis adds costs for the caching layer but significantly reduces load on the database, helping maintain performance at scale.

■ What trade-offs, if any, did you make between cost and
performance?

- I opted for synchronous replication in RDS to ensure high availability, even though this incurs additional cost. The alternative of using a single-instance RDS would reduce costs but compromise the application's resilience, which could lead to greater losses in case of downtime.

- Using CloudFront and Redis incurs some additional cost, but the trade-off is worth it for the significant improvements in performance and scalability. This reduces the load on the web layer and the database, ensuring that both components are more efficient and perform well under varying traffic.

■ Discuss any cost implications of your scaling policies and how they
adapt to varying traffic loads.

While I can't provide precise figures without knowing the exact traffic patterns, I would implement the following scaling policies to keep costs under control:

- Target Tracking Scaling: This policy will automatically adjust the number of tasks for computing instances (Fargate) based on usage. By scaling the application dynamically according to traffic, we avoid over-provisioning, which helps keep costs low.

- Log Retention Policy in CloudWatch: To further reduce costs, I would apply a short log retention policy in CloudWatch. Logs would be transferred to S3 with its own retention policy, ensuring that unnecessary data is not stored for long periods, reducing storage costs.

- Reserved Instances for RDS: To manage database costs effectively while ensuring high availability, I would purchase Reserved Instances for RDS. This would give us predictable pricing and lower costs over time, while still allowing us to handle variable traffic loads efficiently.
