---
title: "Infrastructure with CDK"
date: 2026-11-20
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---
#### Overview

The infrastructure is written in **AWS CDK (Python)** under `infra/` and split into 5 small stacks so each part can be deployed and reviewed on its own. CDK synthesizes **AWS CloudFormation** templates, so every change can be previewed with `cdk diff`.

1. [Bootstrap and data stack](5.4.1-data-stack/): S3, ECR, Lambda container images
2. [Daily flow](5.4.2-daily-flow/): Scheduler → ingest → predict → DynamoDB
3. [API, authentication and web](5.4.3-api-cognito/): API Gateway, Cognito, Amplify Hosting

#### Deployment order

When a change touches several stacks, deploy in this order: `CpawsServing` (tables) → `CpawsData` (Lambda, Scheduler) → `CpawsOps` (alarms) → `CpawsApi` (routes). A resource must exist before anything that reads it. `CpawsAuth` is deployed once, before `CpawsApi`.

#### `-c` parameters are not remembered

`infra/app.py` reads parameters with `try_get_context`. If a deployment **does not pass** a parameter, its value falls back to the default in `app.py`. Two parameters must be passed on **every** deployment:

| Stack | Parameter | If you forget it |
|---|---|---|
| `CpawsOps` | `-c alarm_email=<ALARM_EMAIL>` | The email subscription is removed and alarms stop sending email |
| `CpawsApi` | `-c allowed_origin=<WEB_ORIGIN>` | CORS falls back to `http://localhost:5173` and the web app on Amplify can no longer call the API |

Image tags (`ingest_image_tag`, `predict_image_tag`) and the model version are recorded as defaults in `app.py` through a PR. That way every deployment produces the same configuration.

#### Cost and security principles

- **No resources billed by the hour:** no inference endpoint, no NAT Gateway, no OpenSearch.
- **No Lambda runs inside a VPC.** Ingest calls Yahoo and EIA over the public internet, so no NAT Gateway is needed.
- **Each Lambda has its own least-privilege IAM role.** For example, ingest only has `s3:GetObject` and `s3:PutObject` on `raw/*` and `clean/*`, `s3:ListBucket` limited to these two prefixes, `ssm:GetParameter` on one parameter, logging through the managed policy `AWSLambdaBasicExecutionRole`, and permission to invoke the predict Lambda through Lambda Destinations; the Scheduler may only call `lambda:InvokeFunction` on the ingest function.
- **Tag every resource** with `Project`, `Owner`, `Env` to filter cost.
- **IAM and infrastructure changes go through a Pull Request**, reviewed by the security owner before merge. Deploy only merged commits. Until 17/11/2026 the team deployed from branch `developer`; the submitted version is deployed from branch `production` after a release PR and a tag (`v1.0.0` on 22/11, `v1.0.1` on 23/11/2026). Fixes land on `developer` first and ship with the next release.
- **CI does not deploy.** GitHub holds no AWS permissions; deployments are run by hand from the infrastructure owner's machine, one stack at a time.
