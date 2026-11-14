---
title: "Week 5 Worklog"
date: 2026-10-12
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### OBJECTIVES IN WEEK 5:
- Host a static website on S3 and CloudFront.
- Learn the AWS security services and control cost with IAM policies.
- Learn the AWS database services and practice Amazon RDS.
- Add CI to the project repository and review the first infrastructure and API pull requests.

### TASKS OF WEEK 5:

| Day | Task | Start Date | End Date | References |
|:---:|---|:---:|:---:|---|
| **Mon** | **- Practice the lab: Static website hosting with Amazon S3**<br>+ Create an S3 bucket and upload the website files<br>+ Enable static website hosting, configure Block Public Access and public objects, test the website<br>+ Speed up the website with Amazon CloudFront: block public access to S3, configure and test CloudFront<br>+ Bucket versioning, moving objects, cross-Region replication<br>+ Clean up, notes and best practices<br>**- Team project: add CI to the repository**<br>+ Add a GitHub Actions workflow running pytest and ruff on every push and pull request<br>+ Review and merge PR #1 (CEEMDAN cache and Lambda ingest) | 12/10/2026 | 12/10/2026 | [Start With Amazon S3](https://000057.awsfcaj.com/)<br>[Lambda container images](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html) |
| **Tue** | **- Study Module 05: Security Services on AWS**<br>+ 05-01: Shared Responsibility Model<br>+ 05-02: AWS Identity and Access Management<br>+ 05-03: Amazon Cognito<br>+ 05-04: AWS Organizations<br>+ 05-05: AWS IAM Identity Center<br>+ 05-06: AWS Key Management Service<br>+ 05-07: AWS Security Hub | 13/10/2026 | 13/10/2026 | [Administering access rights with IAM](https://000002.awsfcaj.com/)<br>[Module 05-01](https://youtu.be/tsobAlSg19g?si=-xSAVT8MZReV10RP) |
| **Wed** | **- Practice the lab: Cost and resource governance with IAM**<br>+ Preparation: create an IAM group and an IAM user<br>+ Limit by Region: create the policy, attach it to the group, check that it works<br>+ Limit by EC2 instance family<br>+ Limit by instance size<br>+ Limit by EBS volume type<br>+ Clean up resources | 14/10/2026 | 14/10/2026 | [Cost and Usage Governance](https://000064.awsfcaj.com/) |
| **Thu** | **- Study Module 06: Database Services on AWS**<br>+ 06-01: Database concepts review<br>+ 06-02: Amazon RDS and Amazon Aurora<br>+ 06-03: Amazon Redshift and Amazon ElastiCache<br>**- Team meeting: AWS account, IAM access and data lake**<br>+ Read ADR-0006 (AWS account, IAM access and credits)<br>+ Go through the CDK stack CpawsData (S3 data lake and ECR repository for the ingest image) | 15/10/2026 | 15/10/2026 | [Get Started with Amazon RDS](https://000005.awsfcaj.com/)<br>[Module 06-01](https://youtu.be/OOD2RwWuLRw?si=gUgPzH3nVghQiDQC) |
| **Fri** | **- Practice the lab: Database basics with Amazon RDS**<br>+ Preparation: create the VPC, EC2 and RDS security groups, DB subnet group<br>+ Create an EC2 instance and an RDS database instance<br>+ Deploy the application and connect it to RDS<br>+ Backup and restore<br>+ Clean up resources<br>**- Team review: merge infrastructure and API contract PRs**<br>+ Review and merge PR #4 (CEEMDAN seeds and cross-checks), PR #5 (CpawsData infrastructure) and PR #6 (API contract) | 16/10/2026 | 16/10/2026 | [Get Started with Amazon RDS](https://000005.awsfcaj.com/)<br>[API Gateway REST API](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-rest-api.html) |

### ACHIEVEMENTS IN WEEK 5:
1. Hosted a static website on S3, put CloudFront in front of it and tried versioning and cross-Region replication.
2. Completed Module 05: Shared Responsibility Model, IAM, Cognito, Organizations, Identity Center, KMS, Security Hub.
3. Wrote IAM policies that limit Regions, instance families, sizes and EBS types to keep costs under control.
4. Completed Module 06 and deployed an RDS database with backup and restore.
5. Added the pytest + ruff CI workflow to the team repository and reviewed/merged the first infrastructure and API contract PRs.
