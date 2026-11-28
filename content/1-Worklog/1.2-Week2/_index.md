---
title: "Week 2 Worklog"
date: 2026-09-21
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:
- Practice access management with AWS IAM (users, groups, roles).
- Build a VPC with its subnets, route tables, gateways and security layers.
- Connect AWS with a simulated on-premises network through AWS Site-to-Site VPN.
- Form the CPAWS team, and hold a team project meeting to brainstorm and pick a direction for the project.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| 2 | **- Went to the AWS office for the first time**<br>**- Practice the lab: Access Management with AWS IAM**<br>+ IAM groups, users, policies and roles<br>+ Create an Admin group and an Admin user, sign in as the Admin user<br>+ Create an Admin role and an OperatorUser<br>+ Switch roles: allow OperatorUser to switch role, sign in and switch<br>+ Clean up resources | 21/09/2026 | 21/09/2026 | [Administering access rights with IAM](https://000002.awsfcaj.com/) |
| 3 | **- Meet and form the team**<br>+ Meet the teammates and form the five-person CPAWS team<br>**- Practice the lab: Amazon VPC and AWS Site-to-Site VPN (part 1)**<br>+ Amazon VPC components: subnets, route tables, Internet Gateway, NAT Gateway<br>+ Network security: security groups, network ACLs, VPC Resource Map<br>+ Prepare the environment:<br>* Create the VPC, subnets, Internet Gateway and route table<br>* Create security groups<br>* Enable VPC Flow Logs | 22/09/2026 | 22/09/2026 | [Amazon VPC and AWS Site-to-Site VPN Workshop](https://000003.awsfcaj.com/) |
| 4 | **- Practice the lab: Amazon VPC and AWS Site-to-Site VPN (part 2)**<br>+ Deploy Amazon EC2 instances in the VPC:<br>* Create EC2 instances and test connectivity<br>* Create a Multi-AZ NAT Gateway<br>* Use Reachability Analyzer<br>* Configure an EC2 Instance Connect Endpoint<br>* Connect with AWS Systems Manager Session Manager<br>* Set up CloudWatch monitoring and alerting | 23/09/2026 | 23/09/2026 | [Amazon VPC and AWS Site-to-Site VPN Workshop](https://000003.awsfcaj.com/) |
| 5 | **- Practice the lab: Amazon VPC and AWS Site-to-Site VPN (part 3)**<br>+ Create the VPN environment: a separate VPC and an EC2 instance acting as the customer side<br>+ Configure the VPN connection:<br>* Create a Virtual Private Gateway and a Customer Gateway<br>* Create the VPN connection<br>* Configure the Customer Gateway<br>* Customize the AWS VPN tunnel and advanced VPN options | 24/09/2026 | 24/09/2026 | [Amazon VPC and AWS Site-to-Site VPN Workshop](https://000003.awsfcaj.com/) |
| 6 | **- Practice the lab: Amazon VPC and AWS Site-to-Site VPN (part 4)**<br>+ VPN troubleshooting with the lab guide and the official AWS guide<br>+ Clean up resources<br>**- Team project meeting: brainstorm, analyze and research to select a suitable idea**<br>+ The CPAWS team (5 members) agreed on how to work and the communication channels<br>+ Discussed the direction proposed by Khải: forecasting prices of commodities traded both globally and in Vietnam (gold, silver, crude oil) | 25/09/2026 | 25/09/2026 | [Amazon VPC and AWS Site-to-Site VPN Workshop](https://000003.awsfcaj.com/)<br>[World Bank Commodity Markets](https://www.worldbank.org/en/research/commodity-markets) |

### Week 2 Achievements:
1. Went to the AWS office for the first time; created IAM users, groups and roles and practiced switching roles.
2. Built a VPC with subnets, route tables, Internet Gateway and NAT Gateway, and secured it with security groups and network ACLs.
3. Launched EC2 instances in the VPC and connected to them through Session Manager and EC2 Instance Connect Endpoint.
4. Set up a Site-to-Site VPN between AWS and a simulated on-premises network and troubleshot it with the AWS guide.
5. Formed the CPAWS team (22/09) and joined the first team project meeting: agreed on how to work and discussed the commodity price forecasting direction. On Saturday 26/09 I also attended the Buildathon kickoff (Event 1).
