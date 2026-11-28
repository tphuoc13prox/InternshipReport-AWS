---
title: "Week 3 Worklog"
date: 2026-09-28
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:
- Set up hybrid DNS between AWS and on-premises with Route 53 Resolver.
- Connect several VPCs with VPC Peering and Transit Gateway, and go deeper into AWS networking.
- Attend the AWS Cloud and AI Day Hanoi watch party in Ho Chi Minh City and the fireside chat with Dr. Werner Vogels.
- Hold a team project meeting to finalize the idea, fix the topic and split the roles.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| Mon | **- Practice the lab: Hybrid DNS with Route 53 Resolver**<br>+ Preparation: create a key pair, launch the CloudFormation template, configure security groups<br>+ Connect to the Remote Desktop Gateway (RDGW)<br>+ Deploy AWS Managed Microsoft AD<br>+ Set up DNS:<br>* Create the Route 53 Outbound Endpoint and Resolver Rules<br>* Create the Route 53 Inbound Endpoints<br>* Test name resolution in both directions<br>+ Clean up resources | 28/09/2026 | 28/09/2026 | [Set up Hybrid DNS with Route 53 Resolver](https://000010.awsfcaj.com/) |
| Tue | **- Attend Event 2: AWS Cloud and AI Day Hanoi - Watch Party: Ho Chi Minh City**<br>**- Team project meeting: finalize the idea and add planned features**<br>+ Finalized the CPAWS topic: a web platform forecasting gold and silver (converted from the GLD/SLV ETFs), Brent and WTI prices 1–7 days ahead, displayed in USD and VND<br>+ Added a Vietnamese chatbot on Amazon Bedrock that explains forecasts with cited sources<br>+ Team roles assigned: I am in charge of News & Chatbot (news labeling, chatbot) | 29/09/2026 | 29/09/2026 | [Event 2](../../4-eventparticipated/4.2-event2/)<br>[Amazon Bedrock](https://aws.amazon.com/bedrock/) |
| Wed | **- Practice the labs: VPC Peering and AWS Transit Gateway**<br>+ VPC Peering: launch the CloudFormation template, create security groups and EC2 instances, update network ACLs, create the peering connection, enable cross-peer DNS, clean up<br>+ Transit Gateway: create the Transit Gateway, its attachments and route tables, add Transit Gateway routes to the VPC route tables, clean up | 30/09/2026 | 30/09/2026 | [Setting up VPC Peering](https://000019.awsfcaj.com/)<br>[Set up AWS Transit Gateway](https://000020.awsfcaj.com/) |
| Thu | **- Practice the lab: AWS Networking and Content Delivery (part 1)**<br>+ Preparation: deploy the VPCs, accept the Cisco CSR agreement<br>+ VPC components deep dive<br>+ Transit Gateway and Site-to-Site VPN:<br>* Access the Cisco CSR router<br>* Set up the Site-to-Site VPN and add an ECMP path<br>* Transit Gateway routing<br>+ Route 53 DNS endpoints and private hosted zones: deploy and test DNS | 01/10/2026 | 01/10/2026 | [AWS Networking and Content Delivery](https://000092.awsfcaj.com/) |
| Fri | **- Attend Event 3: Fireside Chat with Dr. Werner Vogels (morning)**<br>+ Lessons from Amazon's operations, the story behind Dynamo and DynamoDB, the "Renaissance Developer"<br>**- Practice the lab: AWS Networking and Content Delivery (part 2)**<br>+ VPC endpoints for AWS services: deploy and check a VPC endpoint<br>+ VPC endpoint services (AWS PrivateLink)<br>+ VPC Peering: peering request, routing configuration, verification<br>+ Transit Gateway Network Manager: set up Network Manager, configure sites, Network Insights<br>+ Clean up resources | 02/10/2026 | 02/10/2026 | [AWS Networking and Content Delivery](https://000092.awsfcaj.com/)<br>[Event 3](../../4-eventparticipated/4.3-event3/) |

### Week 3 Achievements:
1. Set up hybrid DNS with Route 53 Resolver inbound/outbound endpoints and resolved names in both directions.
2. Connected VPCs with VPC Peering and Transit Gateway, and practiced VPC endpoints, PrivateLink and Network Manager.
3. Attended the [AWS Cloud and AI Day Hanoi watch party in Ho Chi Minh City](../../4-eventparticipated/4.2-event2/), on Tuesday 29/09, a day spent on the event and a team project meeting, so it has no study entry; on the morning of 02/10 I also joined the [fireside chat with Dr. Werner Vogels](../../4-eventparticipated/4.3-event3/).
4. Fixed the CPAWS topic (commodity price forecasting with a chatbot) with the team and took the News & Chatbot part.
