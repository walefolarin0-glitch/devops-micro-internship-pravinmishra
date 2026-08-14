# Assignment 5 — Deploy a Highly Available Two-Tier Application on AWS (VPC + ALB + ASG + Multi-AZ RDS)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will design and deploy a highly available two-tier web application on AWS: highly available networking across two Availability Zones, an Application Load Balancer, an Auto Scaling Group for the web tier, and a private Multi-AZ RDS database. You must prove high availability with real failure tests.

---

# Task 1 — Create HA Networking (VPC + 4 Subnets + IGW + NAT + Route Tables)

## Goal

Build a VPC (10.0.0.0/16) with two public and two private subnets across two Availability Zones, an Internet Gateway, a NAT Gateway, and the matching public/private route tables.

### Evidence

#### Screenshot 1 — VPC details showing CIDR 10.0.0.0/16

![screenshot](screenshots/a1.png)

---

#### Screenshot 2 — Subnets list showing four subnets and their Availability Zones

![screenshot](screenshots/a2.png)

---

#### Screenshot 3 — Public route table showing the Internet Gateway route and both public-subnet associations

![screenshot](screenshots/a3.png)

---

#### Screenshot 4 — Private route table showing the NAT Gateway route and both private-subnet associations

![screenshot](screenshots/a4.png)

---

#### Screenshot 5 — NAT Gateway status showing Available and the Elastic IP

![screenshot](screenshots/a5.png)

---

# Task 2 — Create Security Groups (ALB, EC2, RDS) with Least Privilege

## Goal

Create `ha-alb-sg` (HTTP public), `ha-web-sg` (HTTP only from `ha-alb-sg`, SSH from your IP), and `ha-db-sg` (database port only from `ha-web-sg`).

### Evidence

#### Screenshot 6 — ALB Security Group inbound rules

![screenshot](screenshots/a6.png)

---

#### Screenshot 7 — EC2 Security Group inbound rules showing the ALB Security Group reference and SSH from your IP

![screenshot](screenshots/a7.png)

---

#### Screenshot 8 — RDS Security Group inbound rule showing the database port allowed only from the EC2 Security Group

![screenshot](screenshots/a8.png)

---

# Task 3 — Deploy Database Tier (RDS Multi-AZ in Private Subnets)

## Goal

Launch a private, Multi-AZ RDS database (MySQL or PostgreSQL) using the private DB Subnet Group and `ha-db-sg`.

### Evidence

#### Screenshot 9 — RDS summary showing Multi-AZ = Yes and Publicly accessible = No

![screenshot](screenshots/a9.png)

---

#### Screenshot 10 — RDS connectivity section showing the DB Subnet Group and Security Group

![screenshot](screenshots/a10.png)

---

# Task 4 — Build a Launch Template (User Data Installs App + Connects to DB)

## Goal

Create a Launch Template whose user data installs the web-server runtime, deploys the application, configures the database connection, and starts the required services.

### Evidence

#### Screenshot 11 — Launch Template details showing that user data exists, including a visible snippet

![screenshot](screenshots/a11.png)

---

#### Screenshot 12 — A running instance created from the template showing that the application responds on port 80 through a local test or browser using its public IP

![screenshot](screenshots/a12.png)

---

# Task 5 — Create an Application Load Balancer (ALB) Across 2 Public Subnets

## Goal

Create an internet-facing ALB across both public subnets with an HTTP listener and a healthy instance target group.

### Evidence

#### Screenshot 13 — ALB details showing two public subnets in two Availability Zones

![screenshot](screenshots/a13.png)

---

#### Screenshot 14 — Target group showing at least one healthy target

![screenshot](screenshots/a14.png)

---

# Task 6 — Create Auto Scaling Group (ASG) in 2 Public Subnets

## Goal

Create an Auto Scaling Group from the Launch Template across both public subnets, with desired capacity 2, minimum 2, and maximum 4, registered to the ALB target group.

### Evidence

#### Screenshot 15 — Auto Scaling Group showing desired, minimum, and maximum capacity and the selected subnet Availability Zones

![screenshot](screenshots/a15.png)

---

#### Screenshot 16 — EC2 instances list showing two running instances in different Availability Zones

![screenshot](screenshots/a16.png)

---

# Task 7 — Configure App to Use RDS + Validate Read/Write

## Goal

Confirm the application communicates with the RDS database through the ALB DNS name with at least one read and one write operation.

### Evidence

#### Screenshot 17 — Browser showing the application loaded through the ALB DNS name with the URL visible

![screenshot](screenshots/a17.png)

---

#### Screenshot 18 — Proof of a database write through a UI message or database query output

![screenshot](screenshots/a18.png)

---

# Task 8 — High Availability Tests (Must Do Both)

## Goal

Test A: terminate one web instance and confirm the Auto Scaling Group replaces it automatically without interrupting the ALB.

Test B: simulate an Availability Zone impact (stop, detach, or reduce desired capacity in one AZ) and confirm the application stays available.

### Evidence

#### Screenshot 19 — EC2 showing the terminated instance and the newly launched instance; timestamps are helpful

![screenshot](screenshots/a19.png)

---

#### Screenshot 20 — Target group showing healthy targets after replacement

![screenshot](screenshots/a20.png)

---

#### Screenshot 21 — Evidence that an instance was removed, detached, placed in Standby, or stopped in one Availability Zone

![screenshot](screenshots/a21.png)

---

#### Screenshot 22 — Browser showing that the ALB DNS endpoint still works during the change

![screenshot](screenshots/a22.png)

---

# Task 9 — Architecture and Test-Results Summary

## Goal

Summarize the VPC/subnet layout, the ALB and Auto Scaling Group setup, the private Multi-AZ RDS setup, and the results of both high-availability tests.

### Evidence

#### Screenshot 23 — A simple architecture diagram, which may be hand-drawn, or an AWS console overview showing the components

![screenshot](screenshots/a23.png)

---

### Notes

Summarize the VPC and subnets across the two Availability Zones.

In the first Availability Zone, the public subnet contains the NAT Gateway, while the web subnet hosts the web instances managed by the Auto Scaling Group. The data subnet contains the RDS MySQL primary database. In the second Availability Zone, the same subnet structure is replicated, with a public subnet containing another NAT Gateway, a web subnet hosting additional web instances, and a data subnet containing the RDS MySQL read replica.

This subnet design separates the public, application, and database layers while distributing resources across two Availability Zones for improved availability and resilience.

Summarize the ALB and Auto Scaling Group setup.

The setup uses an internet-facing Application Load Balancer (ALB) named Epicbook-ALB within the VPC. The ALB is deployed across two Availability Zones, using public subnet in us-east-1c and public subnet in us-east-1e, providing availability across both zones. It listens for HTTP traffic on port 80 and forwards 100% of the traffic to the Epicbook target group.

Behind the ALB is the Epicbook-ASG (Auto Scaling Group), which maintains a desired capacity of 2 EC2 instances. The group can automatically scale between 1 and 4 instances, allowing the application capacity to adjust as demand changes. The instances are launched using the Epicbook-App-AMI-ec2 launch template with the specified AMI and c7i-flex.large instance type.

Together, the ALB and ASG provide a basic high-availability and scalability layer: the ALB distributes incoming requests to the application instances, while the Auto Scaling Group maintains the required number of instances and can add or remove instances within the configured limits.

Summarize the private Multi-AZ RDS setup.

The database layer runs across two separate data subnets, one in each Availability Zone — both fully private, with no direct internet access (they'd only reach anything external through the NAT Gateways in their respective AZs, and only inbound traffic from the web tier).

AZ-a's data subnet holds the RDS MySQL Master — this is the primary instance that handles all writes.
AZ-b's data subnet holds a Read Replica — a continuously synced copy of the Master, used to offload read traffic and reduce load on the primary.

Summarize the results of both high-availability tests.

The two high-availability tests produced successful results. The ALB target-group test showed 1 registered target with 1 healthy and 0 unhealthy, confirming that the application target was responding correctly to the load balancer health checks.

The EC2 availability test also showed all three running instances passing their 3/3 status checks, confirming that the compute instances were operational and healthy at the time of testing. Together, the results provide evidence that both the load-balancing layer and EC2 application layer were functioning successfully.

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post about the high-availability build, including the ALB URL (or a redacted screenshot), three to five lines on what you built and how you tested high availability, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/wale-folarin-956b6022a_dmibypravinmishra-devops-agenticai-share-7494020870265491456-upKK/?utm_source=share&utm_medium=member_desktop&rcm=ACoAADl6z1IBZjWVdPX--51VXY7TxU7dXOVzE3c

---

#### Screenshot of LinkedIn post

![screenshot](screenshots/a01.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Do not expose passwords, connection strings, private keys, or account IDs

---

# Completion Checklist

- [ ] Task 1: VPC, four subnets, IGW, NAT Gateway, and route tables created (Screenshots 1–5)
- [ ] Task 2: Least-privilege ALB, EC2, and RDS security groups created (Screenshots 6–8)
- [ ] Task 3: Private Multi-AZ RDS created (Screenshots 9–10)
- [ ] Task 4: Self-configuring Launch Template created and tested (Screenshots 11–12)
- [ ] Task 5: ALB created across both public subnets (Screenshots 13–14)
- [ ] Task 6: Auto Scaling Group running two instances across two AZs (Screenshots 15–16)
- [ ] Task 7: Application verified through the ALB with a database read and write (Screenshots 17–18)
- [ ] Task 8: Both high-availability tests completed (Screenshots 19–22)
- [ ] Task 9: Architecture and test-results summary completed (Screenshot 23 & Notes)
- [ ] LinkedIn post published and URL submitted
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*