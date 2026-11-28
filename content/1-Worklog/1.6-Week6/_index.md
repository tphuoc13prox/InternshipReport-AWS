---
title: "Week 6 Worklog"
date: 2026-10-19
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:
- Practice in-memory caching with Amazon ElastiCache for Redis.
- Build highly available web applications with Auto Scaling Groups, load balancers and RDS.
- Start serverless with Lambda for EC2 cost optimization.
- Hold team project meetings to define and finalize the solution architecture and estimate the cost.
- Write the news-source and Bedrock decisions (ADR-0003, ADR-0004) and prepare the chatbot benchmark.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| Mon | **- Practice the lab: Amazon ElastiCache for Redis (part 1)**<br>+ Preparation: create an access key, install and configure the AWS CLI<br>+ Create a subnet group (Console and AWS CLI)<br>+ Create a cluster with cluster mode disabled, connect to the node, delete the cluster<br>**- Team project meeting: define and research the solution architecture for the web app**<br>+ Chose a scheduled serverless architecture: EventBridge Scheduler -> Lambda ingest -> S3 data lake -> Lambda predict -> DynamoDB -> API Gateway -> web<br>**- Team project: update ADR-0003 and ADR-0004 for the MVP**<br>+ ADR-0003: news and signal sources<br>+ ADR-0004: Region and Bedrock access (Nova 2 Lite for simple questions, Claude Haiku 4.5 for tool use) | 19/10/2026 | 19/10/2026 | [Amazon ElastiCache - Redis](https://000061.awsfcaj.com/)<br>[Bedrock model access](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html) |
| Tue | **- Practice the lab: Amazon ElastiCache for Redis (part 2)**<br>+ Use the AWS SDK to read and write data on ElastiCache:<br>* Create clusters (cluster mode disabled and enabled), check the user group<br>* Connect to the clusters<br>* Set and get strings and hashes<br>* Publish/subscribe and read/write streams<br>+ Clean up resources<br>**- Team project: align the news labeling plan with the MVP ADRs**<br>+ Update the news labeling prompt to match the ADRs | 20/10/2026 | 20/10/2026 | [Amazon ElastiCache - Redis](https://000061.awsfcaj.com/) |
| Wed | **- Practice the lab: Deploy the FCJ Management app with an Auto Scaling Group**<br>+ Preparation: network, EC2 instance, RDS database and seed data, web server, metrics for predictive scaling<br>+ Create a launch template<br>+ Set up the load balancer: target group and Application Load Balancer<br>+ Create the Auto Scaling Group<br>+ Test manual, scheduled and dynamic scaling, read predictive scaling metrics<br>+ Clean up resources<br>**- Team project: finalize ADR-0003**<br>+ Write down the terms of use of each news source | 21/10/2026 | 21/10/2026 | [Deploying FCAJ Management with Auto Scaling Group](https://000006.awsfcaj.com/) |
| Thu | **- Practice the lab: Deploy WordPress on AWS**<br>+ Preparation: VPC and subnets, security groups for EC2 and the database, EC2 and RDS instances<br>+ Install WordPress on EC2 and connect it to RDS<br>+ Clean up resources<br>**- Team project meeting: finalize the solution architecture and budget estimate for the web app**<br>+ Estimated costs with the AWS Pricing Calculator (about 15–35 USD/month, within the 200 USD credit), no 24/7 endpoint or NAT Gateway<br>**- Team project: news labeling benchmark with Amazon Nova**<br>+ News labeling prompt v2, JSON schema for the output, max_tokens limit<br>+ Benchmark cases and the first run report | 22/10/2026 | 22/10/2026 | [Deploying Wordpress on AWS Cloud](https://000021.awsfcaj.com/)<br>[Amazon Nova](https://docs.aws.amazon.com/nova/latest/userguide/what-is-nova.html)<br>[AWS Pricing Calculator](https://calculator.aws/#/) |
| Fri | **- Practice the lab: EC2 cost optimization with AWS Lambda**<br>+ Preparation: VPC, security group, EC2 instance, Slack incoming webhook<br>+ Tag the instances<br>+ Create an IAM role for Lambda<br>+ Create Lambda functions to stop and start instances<br>+ Check the results and clean up<br>**- Team project: MVP demo script and chatbot benchmark**<br>+ Manual evidence review of the labeling results<br>+ MVP demo script for 25/10<br>+ Chatbot benchmark proposal: system prompts, question sets, fixtures<br>+ First benchmark run, then a per-branch benchmark script with automatic checks (finished over the weekend) | 23/10/2026 | 23/10/2026 | [Optimize EC2 cost with Lambda](https://000022.awsfcaj.com/)<br>[Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html) |

### Week 6 Achievements:
1. Created ElastiCache Redis clusters (cluster mode disabled and enabled) and used them from the CLI and the SDK (strings, hashes, pub/sub, streams).
2. Deployed the FCJ Management app with an Auto Scaling Group, ALB and RDS, and installed WordPress on EC2 with RDS.
3. Used Lambda to stop and start tagged EC2 instances on a schedule with Slack notifications.
4. Updated ADR-0003 and ADR-0004 for the MVP, ran the first news labeling benchmark with Amazon Nova and wrote the chatbot benchmark proposal and script.
5. Agreed with the team on the scheduled serverless architecture and a cost estimate of about 15–35 USD/month.
