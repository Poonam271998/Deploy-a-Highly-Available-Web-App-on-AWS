# Deploy a Highly Available Web Application on AWS

## Project Overview

This project demonstrates how to deploy a highly available, scalable, and secure web application architecture on Amazon Web Services (AWS).

The infrastructure uses AWS best practices to ensure:

* High Availability
* Fault Tolerance
* Auto Scaling
* Load Balancing
* Secure Networking
* Cost Optimization

The application is deployed across multiple Availability Zones (AZs) using EC2 instances behind an Application Load Balancer (ALB).

---

# Project Objective

Deploy a simple web application in AWS with:

* Highly Available Architecture
* Auto Scaling EC2 Instances
* Application Load Balancer
* Secure VPC Networking
* Public and Private Subnets
* Security Groups and IAM Roles

---

# Architecture Diagram

```text
                    Internet
                        |
                        v
               +----------------+
               | Application    |
               | Load Balancer  |
               +----------------+
                  /          \
                 /            \
                v              v
        +-------------+  +-------------+
        | EC2 Instance|  | EC2 Instance|
        |  AZ-1       |  |  AZ-2       |
        +-------------+  +-------------+
                \              /
                 \            /
                  v          v
               Auto Scaling Group

        Public Subnets + Private Networking
```

---

# AWS Services Used

| AWS Service               | Purpose                       |
| ------------------------- | ----------------------------- |
| Amazon EC2                | Host the web application      |
| Application Load Balancer | Distribute traffic            |
| Auto Scaling Group        | Automatically scale instances |
| Amazon VPC                | Isolated network environment  |
| Internet Gateway          | Internet connectivity         |
| Route Tables              | Network routing               |
| Security Groups           | Firewall rules                |
| IAM                       | Secure access control         |
| Amazon CloudWatch         | Monitoring and alerts         |

---

# Solution Architecture

The application architecture consists of:

## Networking Layer

* One custom VPC
* Two public subnets across different AZs
* Internet Gateway attached to VPC
* Route table configured for internet access

## Compute Layer

* EC2 instances running Apache/Nginx web server
* Launch Template for EC2 configuration
* Auto Scaling Group for elasticity

## Traffic Management

* Application Load Balancer distributes requests
* Health checks ensure only healthy instances receive traffic

## Security Layer

* Security Groups control inbound/outbound traffic
* IAM roles for secure AWS resource access

---

# Prerequisites

Before starting, ensure you have:

* AWS Account
* IAM user with required permissions
* Basic Linux knowledge
* AWS CLI configured (optional)
* Key Pair for EC2 SSH access

---

# Step-by-Step Deployment

# Step 1: Create a VPC

## VPC Configuration

| Setting    | Value       |
| ---------- | ----------- |
| VPC Name   | webapp-vpc  |
| CIDR Block | 10.0.0.0/16 |

---

# Step 2: Create Public Subnets

Create two public subnets in different Availability Zones.

## Subnet 1

| Setting | Value           |
| ------- | --------------- |
| Name    | public-subnet-1 |
| CIDR    | 10.0.1.0/24     |
| AZ      | ap-south-1a     |

## Subnet 2

| Setting | Value           |
| ------- | --------------- |
| Name    | public-subnet-2 |
| CIDR    | 10.0.2.0/24     |
| AZ      | ap-south-1b     |

Enable:

```text
Auto-assign public IPv4
```

---

# Step 3: Create Internet Gateway

1. Create Internet Gateway
2. Attach it to the VPC

Example Name:

```text
webapp-igw
```

---

# Step 4: Configure Route Tables

Create a route table and add:

| Destination | Target           |
| ----------- | ---------------- |
| 0.0.0.0/0   | Internet Gateway |

Associate the route table with both public subnets.

---

# Step 5: Create Security Groups

## Load Balancer Security Group

Allow:

| Type  | Port | Source   |
| ----- | ---- | -------- |
| HTTP  | 80   | Anywhere |
| HTTPS | 443  | Anywhere |

---

## EC2 Security Group

Allow:

| Type | Port | Source           |
| ---- | ---- | ---------------- |
| HTTP | 80   | Load Balancer SG |
| SSH  | 22   | Your IP          |

---

# Step 6: Launch EC2 Instance

## EC2 Configuration

| Setting        | Value          |
| -------------- | -------------- |
| AMI            | Amazon Linux 2 |
| Instance Type  | t2.micro       |
| Key Pair       | Your Key Pair  |
| Security Group | EC2-SG         |

---

# Step 7: Install Web Server

Connect to the EC2 instance:

```bash
ssh -i key.pem ec2-user@PUBLIC-IP
```

Install Apache:

```bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

Create sample webpage:

```bash
echo "<h1>Highly Available AWS Web App</h1>" | sudo tee /var/www/html/index.html
```

Verify:

```text
http://EC2-PUBLIC-IP
```

---

# Step 8: Create Launch Template

Create a Launch Template with:

* Amazon Linux 2 AMI
* Instance type
* Security Group
* User Data Script

## Sample User Data Script

```bash
#!/bin/bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd

echo "<h1>AWS Highly Available Web Application</h1>" > /var/www/html/index.html
```

---

# Step 9: Create Application Load Balancer

## ALB Configuration

| Setting  | Value                     |
| -------- | ------------------------- |
| Name     | webapp-alb                |
| Scheme   | Internet-facing           |
| Type     | Application Load Balancer |
| Listener | HTTP : 80                 |

Select:

* Both public subnets
* Load Balancer Security Group

---

# Step 10: Create Target Group

## Target Group Settings

| Setting           | Value    |
| ----------------- | -------- |
| Target Type       | Instance |
| Protocol          | HTTP     |
| Port              | 80       |
| Health Check Path | /        |

Register EC2 instances.

---

# Step 11: Create Auto Scaling Group

## ASG Configuration

| Setting           | Value |
| ----------------- | ----- |
| Minimum Instances | 2     |
| Desired Capacity  | 2     |
| Maximum Instances | 4     |

Attach:

* Launch Template
* Target Group
* Public Subnets

---

# Step 12: Configure Scaling Policies

Example Scaling Policy:

| Metric          | Threshold |
| --------------- | --------- |
| CPU Utilization | 70%       |

Actions:

* Scale Out when CPU > 70%
* Scale In when CPU < 30%

---

# Testing High Availability

## Test Load Balancer

Access:

```text
http://ALB-DNS-NAME
```

Expected Result:

* Application loads successfully.
* Requests are distributed across instances.

---

# Test Auto Scaling

Generate load using:

```bash
sudo yum install stress -y
stress --cpu 2 --timeout 300
```

Expected Result:

* CPU increases
* Auto Scaling launches new EC2 instance
* New instance registers with ALB

---

# Monitoring Using CloudWatch

Monitor:

* CPU Utilization
* Network Traffic
* Healthy Hosts
* Unhealthy Hosts
* Request Count

Useful CloudWatch Metrics:

| Service | Metric                  |
| ------- | ----------------------- |
| EC2     | CPUUtilization          |
| ALB     | RequestCount            |
| ASG     | GroupInServiceInstances |

---

# Security Best Practices

## Networking Security

* Use Security Groups instead of open access
* Restrict SSH to specific IP addresses
* Use private subnets for databases
* Enable VPC Flow Logs

## IAM Security

* Use least privilege principle
* Avoid using root account
* Use IAM roles for EC2

## Web Security

* Enable HTTPS using ACM certificates
* Configure WAF if required
* Keep servers patched and updated

---

# High Availability Features

This architecture achieves high availability by:

* Deploying instances across multiple AZs
* Using Load Balancer health checks
* Automatically replacing failed instances
* Using Auto Scaling Groups
* Removing single points of failure

---

# Cost Optimization

Ways this project reduces cost:

* Auto Scaling prevents overprovisioning
* Use t2.micro under Free Tier
* Pay only for actual resource usage
* Scale in during low traffic

---

# Advantages of This Architecture

* Highly Available
* Fault Tolerant
* Scalable
* Secure
* Easy to Maintain
* Production Ready

---

# Future Enhancements

Possible improvements:

* Add RDS Multi-AZ Database
* Deploy using Terraform or CloudFormation
* Use ECS or EKS containers
* Add CloudFront CDN
* Implement CI/CD Pipeline
* Add Route 53 domain management
* Configure HTTPS using ACM
* Add AWS WAF protection

---

# Sample AWS CLI Commands

## Create Security Group

```bash
aws ec2 create-security-group \
--group-name webapp-sg \
--description "Web Application Security Group" \
--vpc-id vpc-id
```

---

## Launch EC2 Instance

```bash
aws ec2 run-instances \
--image-id ami-id \
--instance-type t2.micro \
--security-group-ids sg-id \
--subnet-id subnet-id
```

---

# Troubleshooting

| Problem                       | Solution                          |
| ----------------------------- | --------------------------------- |
| ALB showing unhealthy targets | Verify HTTP service is running    |
| EC2 not accessible            | Check Security Group rules        |
| Auto Scaling not working      | Verify CloudWatch alarms          |
| Website not loading           | Check Apache/Nginx status         |
| SSH connection failed         | Verify key pair and inbound rules |

---

# Project Folder Structure

```text
aws-ha-webapp/
│
├── screenshots/
│   ├── vpc.png
│   ├── alb.png
│   ├── autoscaling.png
│   ├── ec2.png
│
├── scripts/
│   ├── userdata.sh
│
├── architecture/
│   └── architecture-diagram.png
│
├── README.md
```

---

# Conclusion

This project demonstrates how to deploy a highly available and scalable web application on AWS using industry-standard cloud architecture.

By combining Amazon EC2, Application Load Balancer, Auto Scaling Groups, and secure VPC networking, the solution ensures reliability, fault tolerance, scalability, and security for modern web applications.

This architecture forms the foundation for enterprise-grade cloud deployments.

---

# Author

Poonam Langote

---


