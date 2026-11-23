---
title: "Architecture and data flow"
date: 2025-09-09
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Current architecture

![CPAWS current architecture](/images/5-Workshop/CPAWS-architecture-current.png)

*Figure 5.1 – CPAWS architecture as deployed on 13/11/2026. Drawn by Châu Trần Mạnh Tiến (CPAWS team) with draw.io and the official AWS Architecture Icons. Nothing on the diagram is planned-only: all of it is live.*

The whole system lives in one AWS account, Region `ap-southeast-1`, environment `dev`. It is fully serverless: no server runs all day, no Lambda function is placed in a VPC, and there is no NAT Gateway. The numbers below follow the numbered arrows on the diagram.

#### Flow 1 – Daily ingest and forecast (Tuesday to Saturday, 19:15 Vietnam time)

1. **EventBridge Scheduler** fires `cron(15 12 ? * TUE-SAT *)` in UTC and invokes the ingest Lambda asynchronously.
2. **Lambda ingest** (container image) downloads prices for GLD, SLV, BZ=F, CL=F and the USD/VND rate; EIA and NBP data are only used as cross-checks.
3. It writes raw files and a cleaned snapshot `clean/prices/dt=<date>/` to the **S3 data lake**.
4. Whether ingest succeeds or fails, a **Lambda destination** (OnSuccess and OnFailure) invokes the predict Lambda. If one source fails, the assets that did get fresh data are still published.
5. **Lambda predict** reads the snapshot and also computes the shadow challenger S1a, which it writes under `eval/shadow/` on S3 (not shown to users).
6. It writes the published forecast to the DynamoDB table **`cpaws-dev-forecasts`** with a conditional `PutItem`, so a forecast that was already published is never overwritten.

The 19:15 slot is deliberate. The backtests assume that a session's data can only be used from 12:00 UTC the next day, and the schedule follows exactly that assumption, so the live system never uses information a backtest would not have had.

#### Flow 2 – Rolling accuracy (same days, 20:45 Vietnam time)

7. A second schedule, `cron(45 13 ? * TUE-SAT *)`, starts the **accuracy job** Lambda (built from the predict image with its own command).
8. The job pairs published forecasts with the prices that came later, within a 56-day window (`rolling_8w`), for both the champion and S1a.
9. It writes the scores to **`cpaws-dev-accuracy`** with conditional writes.

#### Flow 3 – User-facing services

10. Users open the web app on **AWS Amplify Hosting** (React + Vite).
11. They sign in with **Amazon Cognito** and receive an ID token.
12. The browser calls **API Gateway** (REST, stage `dev`) over HTTPS with the ID token. A Cognito authorizer protects every route except `/health`.
13. **Lambda forecasts-api** reads `cpaws-dev-forecasts` for `GET /forecasts/{asset}`. Since release v1.0.0 (22/11) it also reads a short price history from S3 (`serving/price-history`) for the 2-year chart on the web; this arrow is newer than the diagram.
14. **Lambda chat-api** (`POST /chat`) first counts the question against a quota in **`cpaws-dev-chat-usage`**.
15. It then calls **Amazon Bedrock** (Claude Haiku 4.5, Converse API) with two tools: `get_forecast` reads the same forecast table as the web page, and `get_news` reads hand-labelled Fed/EIA news packaged with the Lambda.
16. **Lambda accuracy-api** serves `GET /accuracy/{asset}` from `cpaws-dev-accuracy`.

#### Operations and security

17. CloudWatch alarms send email through **Amazon SNS**.
18. A developer pushes code to GitHub (CI only: lint, tests, web build) and deploys the CDK stacks and container images from a laptop.

The operations panel also shows IAM roles with least privilege, SSM Parameter Store for the EIA key, Amazon ECR for the two images, AWS Budgets, and AWS CDK → CloudFormation with five stacks.

#### AWS services and why we chose them

| Service | Role in CPAWS | Why this choice |
|---|---|---|
| Amazon EventBridge Scheduler | Starts ingest at 19:15 and the accuracy job at 20:45 | Prices change once a day; a schedule is enough and costs nothing between runs |
| AWS Lambda (container image) | Ingest, predict, accuracy job | The heavy Python libraries fit in an image (up to 10 GB); pay per run; the runs take seconds |
| AWS Lambda (zip, Python 3.12) | forecasts-api, accuracy-api, chat-api | Short request/response work, scales by itself |
| Amazon ECR | Stores the ingest and predict images | Images are tagged with the short commit SHA, which makes rollback a one-line change |
| Amazon S3 | Data lake: raw, cleaned snapshots, shadow and evaluation files | Cheap durable storage; versioning protects against overwrites |
| Amazon DynamoDB (on-demand) | Forecasts, news labels, chat quota, accuracy | Reads by key are fast; on-demand billing fits low, uneven traffic |
| Amazon API Gateway (REST) | One API for forecasts, accuracy and chat | Native Cognito authorizer and stage throttling |
| Amazon Cognito | User pool for the web app | Managed sign-in; the API only accepts valid ID tokens |
| Amazon Bedrock | Claude Haiku 4.5 for the chatbot | No GPU to run; tool use keeps numbers tied to the database |
| AWS Amplify Hosting | Serves the single-page web app | Static hosting with rewrites for SPA routes |
| Amazon CloudWatch + Amazon SNS | Logs (1-month retention), metric filters, 6 alarms, email | Failures reach the team without anyone watching a console |
| AWS Systems Manager Parameter Store | The EIA key as `SecureString` | Only one secret, so Secrets Manager is not needed |
| AWS Budgets + Cost Anomaly Detection | Cost alerts | Early warning for unexpected spend |
| AWS CDK / CloudFormation | Five stacks | The whole system can be reviewed in a pull request and recreated or removed with commands |

What we decided **not** to use: Step Functions and SageMaker (there is no trained model to retrain; see [5.11](../5.11-lessons-learned/)), always-on endpoints, a VPC with NAT Gateway, OpenSearch, Bedrock Knowledge Bases, and Bedrock Guardrails (not enabled, see [5.5.2](../5.5-api-and-chatbot/5.5.2-bedrock-chatbot/)).

#### Well-Architected review

**Operational Excellence**
+ All infrastructure is code (five CDK stacks); every change goes through a pull request with at least one approval and CI (`ruff`, `pytest` with CDK synth tests, web type-check/test/build).
+ Six alarms email the team, and the team checks each 19:15 run by hand.
+ Every risky change has a one-command rollback (section [5.7](../5.7-monitoring-and-testing/)).
+ Design decisions are written as ADRs in the repository.

**Security**
+ The API only accepts Cognito ID tokens; throttling is 10 requests/second with a burst of 20; CORS allows a single web origin.
+ Each Lambda role has only the actions it needs. For example the chat Lambda can `Query` the forecast table, `UpdateItem`/`GetItem` on its quota table and `bedrock:InvokeModel` on the exact model and profile ARNs, and nothing for streaming.
+ The only secret lives in SSM Parameter Store; members use MFA and short-term CLI credentials; pre-commit hooks scan for secrets because the repository is public.
+ The S3 bucket blocks public access and enforces SSL. Chatbot answers and error messages never include account IDs, ARNs, table names, model IDs, the system prompt or stack traces.

**Reliability**
+ Predict and the accuracy job use conditional writes, so a re-run cannot corrupt published data. Predict has 0 retries and ingest has 1, to avoid hammering the price source when it is down.
+ Ingest failure still triggers predict; a broken shadow model cannot block the champion.
+ The S3 bucket is versioned (old versions kept 30 days), and the forecast table and user pool are `RETAIN`.
+ The chatbot computes its own deadline from API Gateway's 29-second limit and returns a clean `504` instead of being cut off.

**Performance Efficiency**
+ Forecasts are computed once a day in batch and read by key, so opening a page never runs a model.
+ Measured runs: predict about 3.3 s using about 260 MB of its 1,024 MB; the accuracy job 30.3 s; chatbot answers took 4.0 s at the median and 5.7 s at p90 on the benchmark.

**Cost Optimization**
+ Nothing is billed by the hour: no endpoint, no NAT Gateway, no OpenSearch. DynamoDB is on-demand.
+ Logs are kept 1 month; ECR keeps 5 images per repository.
+ The chatbot quota (20 questions per user per day, 100 per day in total) caps Bedrock spend. Details in [5.8](../5.8-cost/).

**Sustainability** (recorded, not scored)
+ Compute only runs a few minutes a day, and nothing sits idle waiting for traffic.
