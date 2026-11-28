---
title: "Week 9 Worklog"
date: 2026-11-09
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives:
- Hold team project meetings to build the benchmark and decide the production model.
- Learn advanced AWS CDK: ECS, Application Load Balancer, API Gateway, Lambda, S3 and nested stacks.
- Monitor network and application metrics with VPC Flow Logs, Grafana and CloudWatch.
- Check citation links in chatbot answers and finish prompt v4.
- Prepare the Q&A script, report chapter and slides for the chatbot part of the defense.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| Mon | **- Team project meeting: build the benchmark and evaluate the model**<br>+ Statistical benchmark: causal 252-session empirical quantiles (`baseline-q252`)<br>+ Khải and Long presented the comparison of DLinear MVP against the benchmark on validation 2019 using `pinball_skill`<br>**- Team project: citation link check**<br>+ Recap of the internal chatbot trial on 08/11<br>+ Check citation links against get_news items (log in Lambda, fail in the benchmark)<br>+ Finalize Haiku prompt v4 after the dev rerun, per-set thresholds | 09/11/2026 | 09/11/2026 | [DLinear Paper](https://arxiv.org/abs/2205.13504)<br>[Lambda logging](https://docs.aws.amazon.com/lambda/latest/dg/python-logging.html) |
| Tue | **- Practice the labs: VPC Flow Logs and Grafana on AWS**<br>+ VPC Flow Logs: enable flow logs and monitor the network infrastructure, clean up<br>+ Grafana: create the VPC, subnet, security group and EC2 instance, create the IAM user and role, install Grafana, build monitoring dashboards, clean up<br>**- Team project: benchmark rules for v4**<br>+ Close the router retry option in the router proposal<br>+ Require every policy question to pass in the official set<br>+ Review the rolling accuracy job PR (#89); the chatbot Lambda with the citation log was deployed this day | 10/11/2026 | 10/11/2026 | [Monitoring network infrastructure](https://000074.awsfcaj.com/)<br>[Getting started with Grafana basic](https://000029.awsfcaj.com/) |
| Wed | **- Practice the lab: Amazon CloudWatch Workshop**<br>+ CloudWatch metrics: viewing metrics, search expressions, math expressions, dynamic labels<br>+ CloudWatch Logs, Logs Insights and metric filters<br>+ CloudWatch alarms<br>+ CloudWatch dashboards<br>+ Clean up resources<br>**- Team project: final Q&A script for the chatbot defense**<br>+ Q&A script and a check that the demo questions do not overlap with the benchmark sets | 11/11/2026 | 11/11/2026 | [AWS CloudWatch Workshop](https://000036.awsfcaj.com/) |
| Thu | **- Team project meeting: decide the production model and write the Lambda predict code**<br>+ The product uses `baseline-q252`, DLinear stays as challenger<br>+ Lambda predict: read clean data, run the model, `normalize_quantiles()`, write results to DynamoDB<br>**- Team project: update the chatbot chapter with the v4 official result**<br>+ Add the official result of prompt v4 to the chatbot chapter<br>+ Rerun the backup questions of the Q&A script and update it | 12/11/2026 | 12/11/2026 | [DynamoDB Query](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.html) |
| Fri | **- Practice the lab: AWS CDK basics - part 2**<br>+ Preparation: IAM role, EC2 server, VS Code environment<br>+ Build ECS, Application Load Balancer and API Gateway with CDK<br>+ Build Lambda and S3 with CDK<br>+ Create nested stacks with CDK<br>+ Clean up resources<br>**- Team project: chatbot slides for the final presentation**<br>+ Slides for the chatbot part of the final presentation<br>+ Refresh the demo news with the latest EIA STEO and weekly report | 13/11/2026 | 13/11/2026 | [CDK Basic - 2](https://000076.awsfcaj.com/) |

### Week 9 Achievements:
1. Monitored network traffic with VPC Flow Logs and built Grafana and CloudWatch dashboards, alarms and Logs Insights queries.
2. Added the citation link check to the chatbot and the benchmark, and finalized Haiku prompt v4.
3. Finished the Q&A script, the chatbot chapter with the v4 official result and the chatbot slides for the final presentation.
4. Agreed with the team to run `baseline-q252` in production and keep DLinear as challenger.
5. Built ECS, ALB, API Gateway, Lambda, S3 and nested stacks with AWS CDK.
6. On Saturday 14/11 I also attended FCAJ x FPTU: Cloud Mastery 1, a workshop on AI in production and Amazon Bedrock AgentCore (Event 5).
