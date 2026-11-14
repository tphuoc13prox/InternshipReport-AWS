---
title: "Week 9 Worklog"
date: 2026-11-09
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### OBJECTIVES IN WEEK 9:
- Monitor network and application metrics with VPC Flow Logs, Grafana and CloudWatch.
- Check citation links in chatbot answers and finish prompt v4.
- Prepare the Q&A script, report chapter and slides for the chatbot part of the defense.

### TASKS OF WEEK 9:

| Day | Task | Start Date | End Date | References |
|:---:|---|:---:|:---:|---|
| **Mon** | **- Team project: citation link check**<br>+ Recap of the internal chatbot trial on 08/11<br>+ Check citation links against get_news items (log in Lambda, fail in the benchmark)<br>+ Finalize Haiku prompt v4 after the dev rerun, per-set thresholds | 09/11/2026 | 09/11/2026 | [Lambda logging](https://docs.aws.amazon.com/lambda/latest/dg/python-logging.html) |
| **Tue** | **- Practice the labs: VPC Flow Logs and Grafana on AWS**<br>+ VPC Flow Logs: enable flow logs and monitor the network infrastructure, clean up<br>+ Grafana: create the VPC, subnet, security group and EC2 instance, create the IAM user and role, install Grafana, build monitoring dashboards, clean up<br>**- Team project: benchmark rules for v4**<br>+ Close the router retry option in the router proposal<br>+ Require every policy question to pass in the official set<br>+ Review the rolling accuracy job PR (#89); the chatbot Lambda with the citation log was deployed this day | 10/11/2026 | 10/11/2026 | [Monitoring network infrastructure](https://000074.awsfcaj.com/)<br>[Getting started with Grafana basic](https://000029.awsfcaj.com/) |
| **Wed** | **- Practice the lab: Amazon CloudWatch Workshop**<br>+ CloudWatch metrics: viewing metrics, search expressions, math expressions, dynamic labels<br>+ CloudWatch Logs, Logs Insights and metric filters<br>+ CloudWatch alarms<br>+ CloudWatch dashboards<br>+ Clean up resources<br>**- Team project: final Q&A script for the chatbot defense**<br>+ Q&A script and a check that the demo questions do not overlap with the benchmark sets | 11/11/2026 | 11/11/2026 | [AWS CloudWatch Workshop](https://000036.awsfcaj.com/) |
| **Thu** | **- Team project: update the chatbot chapter with the v4 official result**<br>+ Add the official result of prompt v4 to the chatbot chapter<br>+ Rerun the backup questions of the Q&A script and update it | 12/11/2026 | 12/11/2026 |  |
| **Fri** | **- Team project: chatbot slides for the final presentation**<br>+ Slides for the chatbot part of the final presentation<br>+ Refresh the demo news with the latest EIA STEO and weekly report | 13/11/2026 | 13/11/2026 |  |

### ACHIEVEMENTS IN WEEK 9:
1. Monitored network traffic with VPC Flow Logs and built Grafana and CloudWatch dashboards, alarms and Logs Insights queries.
2. Added the citation link check to the chatbot and the benchmark, and finalized Haiku prompt v4.
3. Finished the Q&A script, the chatbot chapter with the v4 official result and the chatbot slides for the final presentation.
