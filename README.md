# 🏗️ Scalable Web Application with Auto Scaling and ALB

This project demonstrates the deployment of a secure, scalable, and highly available web application architecture using core AWS services. The solution is designed across multiple Availability Zones for fault tolerance and high performance, incorporating services such as VPC, EC2, Application Load Balancers (ALB), Auto Scaling Groups (ASG), RDS (MySQL), NAT Gateways, Route 53, CloudWatch, and SNS.

---

## 📘 Table of Contents

- [Architecture Diagram](#architecture-diagram)
- [Architecture Components](#architecture-components)
- [Network Configuration Details](#network-configuration-details)
- [Security Considerations](#security-considerations)
- [Scalability and High Availability](#scalability-and-high-availability)
- [Monitoring and Notifications](#monitoring-and-notifications)
- [Conceptual Setup Steps](#conceptual-setup-steps)

---

## 🖼️ Architecture Diagram

![Architecture Diagram](diagram.png)

---

## 🧱 Architecture Components

### 🔹 Route 53
- Managed DNS service for routing user requests to AWS resources.
- Points to the public-facing Web Tier ALB.

### 🔹 VPC
- Region: `us-east-1`
- AZs: `us-east-1a`, `us-east-1b`
- Provides isolated networking environment.

### 🔹 Internet Gateway (IGW)
- Enables communication between instances in public subnets and the internet.

### 🔹 NAT Gateway
- Located in Public Subnet 2 (AZ2).
- Allows outbound internet access for private subnet instances.

### 🔹 Subnets
- **Public Subnet 1 (AZ1)**: `192.168.1.0/24` – Bastion Host  
- **Public Subnet 2 (AZ2)**: `192.168.2.0/24` – NAT Gateway  
- **Private Subnet 1/2**: Web Tier  
- **Private Subnet 3/4**: App Tier  
- **Private Subnet 5/6**: Database Tier

### 🔹 Route Tables
- Public Route Table for public subnets → IGW.
- Private Route Tables for private subnets → NAT Gateway.

### 🔹 Bastion Host
- Secure access to private instances (SSH).
- Located in Public Subnet 1.

### 🔹 Application Load Balancers
- **ALB-Web** (Public): Distributes incoming traffic to Web Tier.
- **ALB-App** (Internal): Routes traffic from Web Tier to App Tier.

### 🔹 Auto Scaling Groups
- **Web Tier ASG**: Hosts EC2 Web Servers (Private Subnet 1/2).
- **App Tier ASG**: Hosts EC2 App Servers (Private Subnet 3/4).

### 🔹 EC2 Instances
- Web Servers: Serve HTTP and forward to App Tier.
- App Servers: Process logic and interact with database.

### 🔹 IAM Role
- Attached to App Servers for secure access to AWS services like RDS.

### 🔹 RDS (MySQL)
- Multi-AZ deployment for high availability.
- Primary in Private Subnet 5; Standby in Subnet 6.

### 🔹 CloudWatch
- Monitoring and logging for all AWS resources.

### 🔹 SNS
- Email notifications for CloudWatch alarms and other AWS events.

---

## 🌐 Network Configuration Details

- **Route 53** resolves domain to ALB-Web.
- **Public Subnets** access internet via IGW.
- **Private Subnets** use NAT Gateway for outbound internet access.
- Traffic Flow:
  - Users → Internet → Route 53 → IGW → ALB-Web → Web Servers
  - Web Servers → ALB-App → App Servers → RDS (MySQL)
  - Private EC2 → NAT Gateway → IGW

---

## 🔐 Security Considerations

- DNS Security with Route 53 (optionally DNSSEC).
- Subnet segregation: public, private web, app, and DB tiers.
- Bastion Host for admin access.
- Security Groups for tier-based traffic control.
- IAM Roles for least-privileged access.
- RDS policies for backup, encryption, and access control.

---

## 🚀 Scalability and High Availability

- Route 53 supports latency-based routing and failover.
- ALBs distribute traffic across multiple AZs.
- ASGs auto-scale EC2 instances based on load.
- RDS Multi-AZ ensures database availability.

---

## 📈 Monitoring and Notifications

- CloudWatch tracks EC2, RDS, ALBs, ASGs, and Route 53.
- Alarms can scale ASG and notify admins.
- SNS configured for critical email notifications.

---

## 🛠️ Conceptual Setup Steps

### 1. **VPC and Subnets**
- Create a VPC and 8 subnets across two AZs:
  - 2 Public (for IGW, Bastion, NAT)
  - 2 Private Web
  - 2 Private App
  - 2 Private DB

### 2. **Internet and NAT Gateway**
- Attach IGW to VPC.
- Launch NAT Gateway in Public Subnet 2 with Elastic IP.

### 3. **Routing**
- Public Route Table → IGW (0.0.0.0/0)
- Private Route Tables → NAT Gateway (0.0.0.0/0)

### 4. **Security Groups**
- Bastion: SSH + HTTP from trusted IPs.
- ALB-Web: HTTP from internet.
- Web Servers: From ALB-Web only.
- ALB-App: From Web Servers only.
- App Servers: From ALB-App only.
- RDS: From App Servers (MySQL port).

### 5. **Bastion Host**
- Launch EC2 (e.g., t2.micro) in Public Subnet 1 with SSH access.

### 6. **IAM and RDS Setup**
- Create IAM Role for App Tier EC2s.
- Launch Multi-AZ MySQL RDS in Private Subnet 5/6.

### 7. **Launch Configurations**
- Create Launch Templates for Web and App Servers.
- Install app/web code and monitoring agents.

### 8. **Load Balancers and ASGs**
- ALB-Web: Targets Web Tier ASG.
- ALB-App: Targets App Tier ASG.
- Auto Scaling Groups configured for each tier across AZs.

### 9. **Monitoring & Alerts**
- CloudWatch Alarms for CPU, RDS, ALB health, etc.
- SNS for alarm notifications (email).

### 10. **DNS**
- Create Hosted Zone in Route 53.
- Create Alias Record pointing to ALB-Web DNS.

---

