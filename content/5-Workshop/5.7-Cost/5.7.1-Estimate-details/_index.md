---
title: "Appendix: estimate details"
date: 2026-11-25
weight: 1
chapter: false
pre: " <b> 5.7.1. </b> "
---
This page lists every assumption and all 34 lines of the cost estimate built by Nguyễn Tiến Long on 24/11/2026 (summarized in section [5.7](../)). The estimate covers one `cpaws-dev-*` environment in Singapore. The figures observed in November include development, testing and demos, because the system has not yet run in production for a full month. This is an estimate, not a bill or a spending limit.

The Pricing Calculator export was taken at 23:57 on 24/11/2026 (UTC+7), with prices as of 23:33 on 23/11/2026, in `BEFORE_DISCOUNTS` mode, 34 lines, all in `ap-southeast-1`. Bedrock and EventBridge Scheduler were estimated in the public Pricing Calculator (links in section 5.7); SSM Parameter Store uses the Standard price list. The CSV exported from the Console contains the full Account ID, so it is not published.

#### 1. Method

Each figure belongs to one of four kinds: **observed** (existing usage and metrics), **configured** (resources and source code), **assumed** (future volume) and **estimated** (Calculator prices). New lines with Historical cost = 0 are not read as a zero bill. Some account-level or region-level Cost Explorer figures cannot yet be separated by the CPAWS tag, so they are used only as proxies, with a note.

Usage measured over d days is scaled to a 30-day month with `U_month = U_observed × 30 / d`, rounded to the unit. For resources that exist all month (MAU, alarms, storage) or a known cron schedule, the estimate uses the configuration and current level instead of scaling mechanically. The yearly cost is 12 times the monthly cost, with no growth, taxes or price changes.

##### 1.1. Application load

| Parameter | Basis | Per month |
|---|---|---:|
| Cognito Essentials MAU | Cost Explorer: 3 in October, 4 from 01–23/11; assumed 5 active users | 5 |
| Forecast API | Assumed 60 requests per day, not measured traffic | 1,800 |
| Accuracy API | Assumed 20 requests per day | 600 |
| Chat API | Assumed 10 questions per day | 300 |
| API Gateway | 1,800 + 600 + 300; health, preflight and retries not added separately | 2,700 |
| Scheduler | 2 schedules × 5 days a week, rounded to 22 runs each | 44 |
| Lambda jobs | 22 ingest + 22 predict + 22 accuracy-job; each ingest triggers one predict | 66 |
| Lambda total | 2,700 API + 66 jobs | 2,766 |
| Bedrock | 7 successful chats observed on 22–23/11, with 13 model calls | about 557 calls |
| Amplify requests | 679 requests in 2 days × 15; only an indicator of hosting load | 10,185 |

The source code limits the chatbot to 20 questions per user per day and 100 per day for the whole system. The estimate's 300 questions per month is far below the 3,000 per month ceiling. The question limit is not a money or token limit either.

##### 1.2. Lambda: measured and scaled to a month

Measurements come from Lambda and CloudWatch on 17–23/11/2026. All functions are x86_64 with 512 MB of ephemeral storage. Run time is observed, not the timeout. Compute = `memory (GB) × run time (seconds) × invocations`.

| Function | RAM (MB) | Avg run time (ms) | Invocations in 7 days | Duration used (s) | Invocations/month | GB-seconds/month |
|---|---:|---:|---:|---:|---:|---:|
| ingest | 1024 | 12610 | 5 | 12.610 | 22 | 277.420 |
| predict | 1024 | 5392 | 5 | 5.392 | 22 | 118.624 |
| accuracy-job | 1024 | 49621 | 5 | 49.621 | 22 | 1091.662 |
| forecasts-api | 256 | 1299 | 71 | 1.299 | 1,800 | 584.550 |
| accuracy-api | 256 | 314 | 48 | 0.314 | 600 | 47.100 |
| chat-api | 512 | 4596 | 6 | 4.652 | 300 | 697.800 |
| **Total** | | | | | **2,766** | **2817.156** |

The chat-api line uses billed duration; the other functions use the average run time as a proxy. Retries, cold-start differences and unusual concurrency are not added. There are only two Scheduler schedules because predict is invoked through a Lambda Destination, not by its own schedule.

##### 1.3. Storage, requests and data transfer

| Item | Basis | Monthly assumption |
|---|---|---|
| S3 storage | About 89.25 MB of data lake + 1.06 MB of CDK assets | 0.10 GB-month |
| S3 Tier 1 / Tier 2 | 218 / 3381 requests on 17–23/11, × 30/7 | 934 / 14490 requests |
| DynamoDB on-demand | 4 tables; inputs saved in the Calculator | 6381 RRU; 11310 WRU; 0.00158 GB-month |
| Amplify storage | 0.0039 GB over 20 active days, × 30/20 | 0.006 GB |
| Amplify outbound | 5,547,694 bytes in 2 days (22–23/11), scaled to 30 days | 0.084 GB |
| ECR storage | 0.748 GB-month over 23 days, × 30/23 | 0.98 GB-month |
| ECR outbound | 0.129 GB over 23 days, × 30/23 | 0.168 GB |
| CloudWatch alarms | 6 existing alarms | 6 standard alarms |
| CloudWatch custom metrics | 2 configured metric filters; billing not yet confirmed | 2 metrics (reserve) |
| CloudWatch GetMetricData / API / log queries | 8 metrics / 56 requests / 0.0001 GB scanned in 23 days, × 30/23 | 11 metrics / 74 requests / 0.00014 GB |
| CloudWatch logs | Stored and vended log volume over 23 days, × 30/23 | 0.00015 GB-month; 0.00028 GB |
| SNS | 9 requests in 23 days; 1 confirmed email subscription | 12 requests / 12 emails (assumed) |
| KMS | 107 requests in 23 days; no customer-managed keys | 140 requests (assumed) |
| SSM | 2 Standard parameters | 2 parameters, 0 USD |

**Data transfer:** the two outbound lines are Amplify 0.084 GB and ECR 0.168 GB, 0.252 GB per month in total (an amount of data, not a speed). S3 recorded no data in or out on 17–23/11. API Gateway and SNS have no separate egress measurement, so the estimate does not claim they are exactly zero. Amplify only hosts a static site with no billable build minutes, so no build, SSR or WAF cost is included.

##### 1.4. Amazon Bedrock: priced by tokens

In the source code, the router sends every question to the `complex` branch, which uses Claude Haiku 4.5 through the global inference profile, with at most 3 tool rounds and 1500 output tokens per call. Nova 2 Lite is in the code but has no usage, so it has no line. The estimate assumes no prompt caching, image input or Guardrails.

| Parameter | Value |
|---|---:|
| CloudWatch 17–23/11 | 15 model calls; 82,198 input / 3,708 output tokens |
| Sample 22–23/11 (matched with chat logs) | 7 successful chats; 13 calls; 71,904 input / 3,191 output tokens |
| Avg model calls per question | 13/7 = 1.857 |
| Avg input tokens per question | 10,272 |
| Avg output tokens per question | 455.857 |
| 300 questions per month | about 557 calls; 3,081,600 input / about 136,757 output tokens |
| Calculator price (Global On-Demand) | 1 USD per million input tokens; 5 USD per million output tokens |
| **Cost** | **3.0816 + 0.684 = 3.7656 USD per month** |

In the public Pricing Calculator, the token total is entered as 1,800 equivalent requests (1 request per minute, 1 hour per day, 1712 input and 76 output tokens per request), not the real number of calls. Output tokens differ slightly because the Calculator only accepts whole numbers. The 7-question sample is small and does not cover every case; longer questions or more tool rounds raise the cost.

##### 1.5. EventBridge Scheduler

The two schedules `cpaws-dev-ingest-daily` and `cpaws-dev-accuracy-daily` are Enabled and run `TUE-SAT`. The accuracy schedule was only created on 11/11, so the estimate does not extrapolate from past runs; it uses 44 runs per month from the current cron. The public Pricing Calculator records 44 invocations per month with a 1 KB payload, within the Free Tier of 14 million, so the cost is 0 USD. Retries beyond normal runs are not included.

#### 2. All 34 estimate lines

Quantities and costs are taken as-is from the Pricing Calculator export, without intermediate rounding; Lambda groups are shortened to the function name.

| Service code | Group / function | Usage type | Operation | Monthly quantity | USD/month |
| --- | --- | --- | --- | --- | --- |
| AWSLambda | ingest | APS1-Lambda-GB-Second | — | 277.42 Lambda-GB-Second | 0.004623676 |
| AWSLambda | ingest | APS1-Request | — | 22 Request | 0.0000044 |
| AWSLambda | accuracy-api | APS1-Lambda-GB-Second | — | 47.1 Lambda-GB-Second | 0.000785002 |
| AWSLambda | accuracy-api | APS1-Request | — | 600 Request | 0.00012 |
| AWSLambda | accuracy-job | APS1-Lambda-GB-Second | — | 1091.662 Lambda-GB-Second | 0.018194403 |
| AWSLambda | accuracy-job | APS1-Request | — | 22 Request | 0.0000044 |
| AWSLambda | chat-api | APS1-Lambda-GB-Second | — | 697.8 Lambda-GB-Second | 0.011630023 |
| AWSLambda | chat-api | APS1-Request | — | 300 Request | 0.00006 |
| AWSLambda | forecasts-api | APS1-Lambda-GB-Second | — | 584.55 Lambda-GB-Second | 0.009742519 |
| AWSLambda | forecasts-api | APS1-Request | — | 1800 Request | 0.00036 |
| AWSLambda | predict | APS1-Lambda-GB-Second | — | 118.624 Lambda-GB-Second | 0.001977071 |
| AWSLambda | predict | APS1-Request | — | 22 Request | 0.0000044 |
| AmazonApiGateway | CPAWS-API-Gateway | APS1-ApiGatewayRequest | ApiGatewayRequest | 2700 APS1-ApiGatewayRequest | 0.011475 |
| AmazonDynamoDB | CPAWS-DynamoDB | APS1-ReadRequestUnits | PayPerRequestThroughput | 6381 ReadRequestUnits | 0.000909292 |
| AmazonDynamoDB | CPAWS-DynamoDB | APS1-TimedStorage-ByteHrs | — | 0.00158 GB-Mo | 0 |
| AmazonDynamoDB | CPAWS-DynamoDB | APS1-WriteRequestUnits | PayPerRequestThroughput | 11310 WriteRequestUnits | 0.0080301 |
| AmazonS3 | CPAWS-S3 | APS1-Requests-Tier1 | — | 934 Requests | 0.00467 |
| AmazonS3 | CPAWS-S3 | APS1-Requests-Tier2 | — | 14490 Requests | 0.005796 |
| AmazonS3 | CPAWS-S3 | APS1-TimedStorage-ByteHrs | — | 0.1 GB-Mo | 0.0025 |
| AWSAmplify | CPAWS-Amplify | APS1-DataStorage | — | 0.006 GigaBytes | 0.000138 |
| AWSAmplify | CPAWS-Amplify | APS1-DataTransferOut | — | 0.084 GigaBytes | 0.0126 |
| AmazonCognito | CPAWS-Cognito | APS1-CognitoEssentialsMAU | CognitoEssentialsOperation | 5 CognitoUserPoolsMAU | 0.075 |
| AmazonECR | CPAWS-ECR | APS1-TimedStorage-ByteHrs | — | 0.98 GB-Mo | 0.098 |
| AWSDataTransfer | CPAWS-ECR | APS1-DataTransfer-Out-Bytes | — | 0.168 GB | 0.02016 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-CW:AlarmMonitorUsage | — | 6 Alarms | 0.6 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-CW:GMD-Metrics | GetMetricData | 11 Metrics | 0.00011 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-CW:MetricMonitorUsage | — | 2 Metrics | 0.6 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-CW:Requests | — | 74 Requests | 0.00074 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-DataScanned-Bytes | StartQuery | 0.00014 GB | 0.00000098 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-TimedStorage-ByteHrs | — | 0.00015 GB-Mo | 0.0000045 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-VendedLog-Bytes | PutLogEvents | 0.00028 GB | 0.000196 |
| AmazonSNS | CPAWS-SNS | APS1-DeliveryAttempts-SMTP | — | 12 Notifications | 0 |
| AmazonSNS | CPAWS-SNS | APS1-Requests-Tier1 | — | 12 Requests | 0 |
| awskms | CPAWS-KMS | ap-southeast-1-KMS-Requests | — | 140 Requests | 0.00042 |
| **Console subtotal** | | | | 34 lines | 1.488255766 |

#### References

- [AWS Pricing Calculator](https://calculator.aws/) and [AWS Well-Architected – Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)
- [Amazon Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/)
- [Amazon EventBridge Pricing](https://aws.amazon.com/eventbridge/pricing/)
- [AWS Systems Manager Pricing](https://aws.amazon.com/systems-manager/pricing/) and [AWS KMS Pricing](https://aws.amazon.com/kms/pricing/)
- Pricing for [Lambda](https://aws.amazon.com/lambda/pricing/), [API Gateway](https://aws.amazon.com/api-gateway/pricing/), [DynamoDB](https://aws.amazon.com/dynamodb/pricing/), [S3](https://aws.amazon.com/s3/pricing/), [Amplify](https://aws.amazon.com/amplify/pricing/), [Cognito](https://aws.amazon.com/cognito/pricing/), [ECR](https://aws.amazon.com/ecr/pricing/), [CloudWatch](https://aws.amazon.com/cloudwatch/pricing/), [SNS](https://aws.amazon.com/sns/pricing/)
- [AWS Budgets](https://aws.amazon.com/aws-cost-management/aws-budgets/) and [AWS Cost Anomaly Detection](https://aws.amazon.com/aws-cost-management/aws-cost-anomaly-detection/)

*Prices as of 23–24/11/2026; price lists and discounts may change afterwards.*
