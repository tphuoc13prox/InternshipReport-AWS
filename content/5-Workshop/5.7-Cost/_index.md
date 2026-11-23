---
title: "Cost and Well-Architected"
date: 2026-11-20
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---
#### Actual cost

All costs are paid from the team's AWS credits (about 200 USD). The figures below come from AWS Billing and Cost Explorer.

| Item | Amount | Notes |
|---|---:|---|
| Actual cost 12/10 – 11/11/2026 | about 5.28 USD | Includes experiments and benchmarks |
| Forecast 12/11 – 29/11/2026 | about 1.14 USD | A forecast, not cost already incurred |
| One official chatbot benchmark run | about 0.73 USD | Real Bedrock calls, 11/11/2026 |

The final figures will be updated after 30/11/2026.

#### Estimate with the AWS Pricing Calculator

Estimate: [PRICING CALCULATOR LINK TO BE ADDED]

The estimate uses the following assumptions, all of which are actual usage or limits configured in the system:

| Service | Monthly usage assumption |
|---|---|
| AWS Lambda | About 22 run days × (ingest + predict + accuracy job), 1,024 MB each; a few thousand API calls |
| Amazon EventBridge Scheduler | About 44 invocations (2 schedules × 22 days) |
| Amazon S3 | A few GB of Parquet and compressed CSV, with versioning |
| Amazon DynamoDB (on-demand) | 4 forecast records per day; reads driven by page views and chatbot questions |
| Amazon ECR | 2 repositories, at most 5 images each |
| Amazon API Gateway (REST) | A few thousand requests |
| Amazon Cognito | Fewer than 50 active users |
| AWS Amplify Hosting | Static web app, a few manual deployments |
| Amazon CloudWatch | Logs kept 1 month, 6 alarms |
| Amazon Bedrock (Claude Haiku 4.5) | About 0.012 USD per question; hard cap of 100 questions per day |

**Bedrock is the item that can grow fastest.** That is why the chatbot requires sign-in, with a limit of 20 questions per user per day and 100 per day for the whole system. At that cap, the chatbot costs at most about 1.2 USD per day.

#### Cost optimization measures

| Measure | Effect |
|---|---|
| Batch forecasting in Lambda instead of a 24/7 inference endpoint | Pay for a few seconds of predict per day instead of 730 hours per month |
| Drop Step Functions and SageMaker AI (ADR-0011) | No orchestration or periodic training cost |
| No Lambda in a VPC, no NAT Gateway | Avoid NAT Gateway hourly and data processing charges |
| Forecasts precomputed, stored in on-demand DynamoDB, read with `Query` and `Limit=1` | Each page view costs one small read |
| NumPy inference in the predict Lambda instead of PyTorch | Small image, fast start, fewer GB-seconds |
| Chatbot question limits and `maxTokens` 1500 | Caps unexpected Bedrock cost |
| ECR keeps 5 images, logs kept 1 month, old S3 versions kept 30 days | Nothing accumulates forever |
| AWS Budgets at 50 USD/month and Cost Anomaly Detection | Early warning of unusual cost |
| `Project`, `Owner`, `Env` tags on every resource | Filter cost by project in Cost Explorer |

#### Review against AWS Well-Architected

| Pillar | Done | Weak points |
|---|---|---|
| **Operational Excellence** | Infrastructure as code (CDK, 5 stacks), `cdk diff` before deploying, CI on every PR, rollback with one deploy command using an old image tag, an ADR for every architecture decision | Manual deployment from one machine, no CD; Amplify by manual zip upload |
| **Security** | IAM users + groups, MFA, short-lived sign-in sessions; least-privilege IAM role per Lambda; S3 public access blocked, SSL enforced; API key in SSM `SecureString`; Cognito authorizer, throttling, single-origin CORS; deletion of evaluation results blocked by explicit deny | Bedrock Guardrails not enabled; the read-only group uses `ReadOnlyAccess`, broader than needed |
| **Reliability** | A failed source still lets the other assets publish; conditional writes make re-runs safe; versioned bucket; forecasts table and User Pool are `RETAIN`; 6 alarms send email | One region, one environment; no alert when the schedule does not run |
| **Performance Efficiency** | Lambda container images chosen for large libraries; predict runs in about 3.3 seconds with 1,024 MB; forecasts are precomputed so the API reads one record | Chat Lambda cold starts add to answer time |
| **Cost Optimization** | No resources billed by the hour; actual cost about 5.28 USD after the first month; Budgets and Cost Anomaly Detection | Bedrock cost not yet broken down by feature |
| **Sustainability** | Runs only when there is a new trading session (Tuesday–Saturday); no idle machines | — |

<!-- ẢNH: Cost Explorer, chi phí theo dịch vụ (che Account ID) -->
<!-- ẢNH: AWS Pricing Calculator, trang tổng của bảng ước tính -->
