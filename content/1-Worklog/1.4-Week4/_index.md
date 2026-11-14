---
title: "Week 4 Worklog"
date: 2026-10-05
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### OBJECTIVES IN WEEK 4:
- Learn Amazon EC2 in depth: instance types, AMI, EBS, user data, Auto Scaling.
- Deploy web applications on EC2 Linux and Windows, and manage resources from the AWS CLI.
- Learn the AWS storage services (Module 04).
- Create the project repository and finish the proposal with the team.

### TASKS OF WEEK 4:

| Day | Task | Start Date | End Date | References |
|:---:|---|:---:|:---:|---|
| **Mon** | **- Study Module 03: Compute VM on AWS**<br>+ 03-01: Compute VM on AWS<br>+ 03-01-01: EC2 instance types<br>+ 03-01-02: AMI, backup, key pair<br>+ 03-01-03: Elastic Block Store (EBS)<br>+ 03-01-04: EC2 Instance Store<br>+ 03-01-05: EC2 user data<br>+ 03-01-06: EC2 metadata<br>+ 03-01-07: EC2 Auto Scaling<br>+ 03-02: EC2 Auto Scaling, EFS/FSx, Lightsail, MGN<br>**- Team project: create the CPAWS-commodity-forecasting repository**<br>+ Create the GitHub repository for the team (initial commit) | 05/10/2026 | 05/10/2026 | [Introduction to Amazon EC2](https://000004.awsfcaj.com/)<br>[Module 03-01](https://youtu.be/-t5h4N6vfBs?si=bS-jetOxckl24fDr) |
| **Tue** | **- Practice the lab: Amazon EC2 (part 1)**<br>+ Preparation: create VPCs and security groups for Linux and Windows<br>+ Launch and connect to a Windows instance and a Linux instance<br>+ Amazon EC2 basics:<br>* Resize an instance<br>* Create an EBS snapshot and a custom AMI, launch an instance from the custom AMI<br>* Regain access when the key pair is lost (Windows via SSM, Linux via user data)<br>* Configure a desktop environment for EC2 Ubuntu 22.04<br>* Archive EBS snapshots and share an AMI | 06/10/2026 | 06/10/2026 | [Introduction to Amazon EC2](https://000004.awsfcaj.com/) |
| **Wed** | **- Practice the lab: Amazon EC2 (part 2)**<br>+ Deploy a Node.js application on Amazon Linux: install the LAMP stack and phpMyAdmin, install Node.js, deploy the app<br>+ Deploy a Node.js application on EC2 Windows: install XAMPP and Node.js, deploy the app<br>+ Resource usage governance with IAM: limit by Region, instance family, instance type, EBS volume type, corporate IP address and time window<br>+ Clean up resources<br>**- Team review: repository structure and final proposal**<br>+ Review the folder layout (proposal, architecture, ADR, defense), Python config and pre-commit checks<br>+ The final proposal is added to the repository | 07/10/2026 | 07/10/2026 | [Introduction to Amazon EC2](https://000004.awsfcaj.com/)<br>[Proposal Sample](https://workshop-sample.awsfcaj.com/2-proposal/) |
| **Thu** | **- Practice the lab: Getting started with the AWS CLI**<br>+ Install and configure the AWS CLI<br>+ Check resources from the CLI<br>+ Use the AWS CLI with Amazon S3, Amazon SNS and IAM<br>+ Use the AWS CLI with VPC and Internet Gateway<br>+ Create an EC2 instance with the AWS CLI<br>+ Troubleshooting and resource cleanup | 08/10/2026 | 08/10/2026 | [Getting Started with the AWS CLI](https://000011.awsfcaj.com/) |
| **Fri** | **- Study Module 04: Storage Services on AWS**<br>+ 04-02: Amazon S3, access points, storage classes<br>+ 04-03: S3 static website and CORS, access control, object keys and performance, Glacier<br>+ 04-04: Snow Family, Storage Gateway, AWS Backup<br>**- Team review: price data ingest pipeline**<br>+ Review the price ingest pipeline (yfinance for GLD, SLV, CL=F, BZ=F, VND=X with EIA cross-checks) and the price EDA<br>+ Read ADR-0005 on price data sources | 09/10/2026 | 09/10/2026 | [Start With Amazon S3](https://000057.awsfcaj.com/)<br>[Module 04-02](https://youtu.be/_yunukwcAwc?si=6HqDHVQJq8Ks9aWh) |

### ACHIEVEMENTS IN WEEK 4:
1. Learned the EC2 building blocks (instance types, AMI, EBS, instance store, user data, metadata, Auto Scaling).
2. Practiced resizing, snapshots, custom AMIs and regaining access to instances without the key pair.
3. Deployed a Node.js app on EC2 Linux (LAMP) and Windows (XAMPP), and limited resource usage with IAM conditions.
4. Created VPC, S3, SNS, IAM and EC2 resources from the AWS CLI.
5. Completed Module 04: S3, storage classes, Glacier, Snow Family, Storage Gateway, AWS Backup.
6. Created the team repository; the final proposal and the first data ingest pipeline were added this week.
