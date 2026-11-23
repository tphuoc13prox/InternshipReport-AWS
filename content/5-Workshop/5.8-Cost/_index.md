---
title: "Cost"
date: 2025-09-09
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

#### Estimate

**AWS Pricing Calculator:** [CẦN BỔ SUNG: link Pricing Calculator của nhóm]

| Service | What is charged | Monthly estimate |
|---|---|---:|
| [CẦN BỔ SUNG: per-service rows from the team's Pricing Calculator estimate] | | |

#### Actual spend so far

Figures recorded by the team on 11/11/2026. They are working numbers for the project, not a closed billing period.

| Item | Amount |
|---|---:|
| Actual cost 12/10 – 11/11 | about 5.28 USD |
| Forecast 12/11 – 29/11 | about 1.14 USD |
| Credit left on 11/11 | about 194.67 USD |

All spending so far is covered by AWS credits. The final per-service figures will be taken from Cost Explorer after 30/11.

#### Why the bill is small

The design avoids anything that is charged by the hour while nobody uses it:

+ **Lambda only runs when needed:** two scheduled runs a day on weekdays, a few seconds for predict and about 30 seconds for the accuracy job, plus API calls.
+ **No always-on endpoint**, no SageMaker, no OpenSearch. The champion is a statistical baseline, so there is nothing to host.
+ **No VPC for Lambda**, so no NAT Gateway, which alone would cost more than the rest of the system.
+ **DynamoDB on-demand**, which fits low and uneven traffic.
+ **Retention limits:** CloudWatch logs kept 1 month, ECR keeps 5 images per repository, old S3 versions expire after 30 days.

#### The one cost that can grow: Bedrock

The chatbot is the only part whose cost grows with users, so it has a hard ceiling:

| Guard | Value |
|---|---|
| Cost per question (Haiku 4.5, measured on the benchmark) | about 0.012 USD |
| Quota per user | 20 questions/day |
| Quota for the whole system | 100 questions/day |
| Worst case | about 1.2 USD/day, about 36 USD/month |
| Stage throttling | 10 requests/s, burst 20 |

The worst case stays under the project budget of 50 USD per month. Requests that fail validation are not counted, and 5xx errors are not refunded, so retrying in a loop cannot raise the cost. The two official prompt v4 benchmark runs together cost about 0.73 USD.

#### Alerts

| Mechanism | Configuration |
|---|---|
| AWS Budgets | 50 USD per month |
| Cost Anomaly Detection | Default monitor, daily check, alert from 100 USD absolute or 40% relative, confirmed email subscription |

Both only warn; neither stops a service when a threshold is crossed. That is why the chatbot quota above is enforced in code.

<!-- IMAGE: Billing console, cost by service for the project period (no account ID visible) -->
[ẢNH: Billing – cost by service, account ID hidden]
