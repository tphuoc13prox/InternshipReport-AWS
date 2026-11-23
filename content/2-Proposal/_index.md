---
title: "Proposal"
date: 2025-09-09
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CPAWS – Commodity Price Forecasting on AWS
## Serverless 1–7 session forecast ranges for gold, silver, Brent and WTI, with a Bedrock chatbot that explains them

### 1. Executive Summary
CPAWS is the project of our five-person team in the First Cloud AI Journey program. We set out to build a website on AWS that, for four commodities (gold, silver, Brent crude and WTI crude) and each horizon of 1 to 7 trading sessions ahead, shows a forecast range instead of a single price, the probability that the price goes up, and how reliable past forecasts have actually been. A Vietnamese chatbot on Amazon Bedrock sits next to the forecasts and explains them in plain language. It only quotes numbers that it reads from the system through tools, and it never gives investment advice.

The plan was built around a serverless design: scheduled Lambda functions collect prices and publish forecasts once a day, results are stored in Amazon S3 and Amazon DynamoDB, and users reach them through Amazon API Gateway, Amazon Cognito and an AWS Amplify web app. The whole infrastructure is written with AWS CDK in one `dev` environment in `ap-southeast-1`. The build window is 7 weeks, from 12/10 to 29/11/2026.

### 2. Problem Statement
### What's the Problem?
Gold, silver and crude oil can move sharply when something big happens: a conflict, an interest-rate decision, an OPEC+ output decision. In Vietnam these prices matter to ordinary people, because domestic gold and silver prices track world prices and crude oil drives retail fuel prices. Anyone can look up today's price, but three things are still hard:

- **What comes next is unclear.** Forecasts in the news are usually opinions, without numbers and without any record of how accurate they were.
- **Why it moves is unclear.** News arrives in large, scattered pieces that are not tied to a concrete number.
- **It is hard to relate to your own money.** World prices are quoted in USD per ounce or USD per barrel.

### The Solution
Every trading day after the US session closes, a scheduled job downloads prices, cleans them and stores a snapshot in an S3 data lake. A second function forecasts the **distribution of the log return** over the next 1–7 sessions as seven quantiles and writes the result to DynamoDB. We forecast returns rather than price levels on purpose: tomorrow's price is almost always close to today's, so a price-level error looks small even for a model that has learned nothing. Working with returns forces a model to learn the actual change, and it lets us measure accuracy honestly.

The web app turns the quantiles back into a price range in USD or VND and a probability of going up. Gold and silver are forecast on the GLD and SLV ETFs as proxy series, so their prices on the site are converted from GLD/SLV and are not spot prices. A chatbot built with the Bedrock Converse API answers questions in Vietnamese. It reads forecasts and labelled news through tools instead of inventing numbers.

### Benefits and Return on Investment
- **For users:** a range with a probability gives a more honest picture than a single "target price", and the reliability block shows how well past forecasts held up.
- **For the team:** the project covers a full data and ML product on AWS: ingestion, storage, forecasting, an API with authentication, a web front end, a GenAI chatbot, monitoring and cost control.
- **Cost:** nothing is billed by the hour. There are no servers running all day, no always-on model endpoint and no NAT Gateway, so the bill grows with usage instead of with time. The details are in section 6.
- CPAWS is a learning project with no revenue, so we do not claim a payback period. Its value is a working product that other people can study and reproduce from the CDK code.

### 3. Solution Architecture
The proposal architecture below is the plan our team drew in draw.io at the start of the project. The architecture that is actually deployed is in the [Workshop](../5-workshop/) section, and the differences are listed at the end of this section.

![CPAWS proposal architecture](/images/2-Proposal/CPAWS-architecture.jpg)

*Figure 2.1 – CPAWS architecture in the proposal (drawn by the CPAWS team with draw.io).*

The plan has three parts:

1. **A daily data and forecasting pipeline** triggered by a schedule: collect prices and news, label the news, store everything in S3, run the forecast and save it in DynamoDB.
2. **A weekly retraining loop** orchestrated by AWS Step Functions with SageMaker AI training jobs and a model registry.
3. **User-facing services:** a web app on Amplify, sign-in with Cognito, a REST API on API Gateway and Lambda, and a Bedrock chatbot with Guardrails and a knowledge base.

### AWS Services Used
- **Amazon EventBridge Scheduler:** starts the daily pipeline at a fixed time.
- **AWS Lambda:** collects data, publishes forecasts and serves the API, without servers to manage.
- **Amazon S3:** data lake for raw data, cleaned price snapshots and evaluation files.
- **Amazon DynamoDB:** stores ready-to-read forecasts so the API can answer quickly.
- **AWS Step Functions and Amazon SageMaker AI:** planned for weekly retraining, training jobs and a model registry.
- **Amazon Bedrock:** news labelling and the chatbot (Converse API with tool use, Guardrails, Knowledge Bases).
- **Amazon API Gateway and Amazon Cognito:** a REST API that only signed-in users can call, with request throttling.
- **AWS Amplify Hosting:** hosts the React web app.
- **Amazon CloudWatch and Amazon SNS:** logs, metrics, alarms and email alerts.
- **AWS Secrets Manager:** planned for API keys of the data sources.
- **AWS CDK / AWS CloudFormation:** infrastructure as code so the system can be recreated or removed with one command.

### Component Design
- **Data collection:** daily prices for GLD, SLV, BZ=F and CL=F, the USD/VND rate for display, and news and macro signals from several sources.
- **Data lake:** raw files and cleaned snapshots on S3, one snapshot per run date.
- **Forecasting:** seven quantiles of the log return for h = 1–7 sessions per asset. The proposal chose CEEMDAN + DLinear/LSTM as the main model, with simpler models as baselines.
- **Serving:** the latest forecast per asset is written to DynamoDB, and the API only reads it.
- **Web:** a basic and an advanced mode, prices in USD or VND, and a public record of historical accuracy.
- **Chatbot:** Vietnamese questions and answers about forecasts and related news, with sources, and refusals for out-of-scope questions and investment advice.
- **Operations:** alarms by email, budgets, and CI on GitHub.

**Changes after the proposal.** While building, the team tested each part against the plan and changed what did not hold up. Every change has a written decision (ADR) in the team repository:

| Proposal | Deployed system | Why |
|---|---|---|
| Step Functions → SageMaker training job → model registry, retrained weekly | Two Lambda functions linked by a Lambda destination. The published forecast is the 252-session statistical baseline `baseline-q252`, and a challenger (S1a) runs in shadow mode | 13 configurations of DLinear, CEEMDAN + DLinear, volatility scaling and GJR-GARCH did not beat the baseline on the 2019 validation set, so there is no trained model to retrain |
| Bedrock batch labelling of news + Knowledge Bases | News labelled by hand, packaged as JSON with the chat Lambda | Automatic labelling did not reach the quality threshold |
| Many news and macro sources | Prices and FX from Yahoo Finance (yfinance); EIA and NBP only for cross-checks | Data scope and terms of use |
| Secrets Manager | SSM Parameter Store SecureString | Only one key (EIA) is needed |
| Chatbot with Guardrails | Converse + tool use, Guardrails not enabled | Turning them on after the official benchmark would change the behaviour that was evaluated |
| GitHub Actions CI/CD | CI only; deployment runs from a team member's machine | GitHub does not hold AWS credentials and the repository is public |
| Amplify connected to Git | Manual zip upload | The demo date came first; the domain is kept unchanged for grading |
| — | Added: daily rolling-accuracy job, `GET /accuracy/{asset}`, 6 alarms, chat quota | Needed to measure live forecasts and to control cost |

### 4. Technical Implementation
**Implementation Phases**
The team planned the 7 weeks in four phases:

1. **Data and baseline (from 12/10):** price ingestion, cleaning against exchange calendars, the S3 data lake, and the 252-session baseline as the reference every model must beat.
2. **Model experiments (October to early November):** each experiment had its plan, candidates and pass criterion committed before it was run, and every look at the 2019 validation set was counted.
3. **Product (late October to mid November):** the forecast API, Cognito sign-in, the web app on Amplify, the `POST /chat` chatbot, the rolling-accuracy job and the alarms. Milestones: first automatic forecast on 27/10, chatbot deployed on 02/11, web app on Amplify on 04/11, shadow challenger on 04/11, accuracy schedule enabled on 12/11.
4. **Freeze and hand-in (22/11 to 29/11):** no new features after 22/11; screenshots, rehearsal, report and workshop.

**Technical Requirements**
- Python 3.12 for Lambda code and data processing; AWS CDK (Python) for infrastructure; Docker to build the ingest and predict container images; Node.js with React, TypeScript and Vite for the web app.
- `pandas_market_calendars` for NYSE and CME trading calendars, so that "h sessions ahead" counts real trading sessions.
- Access to Amazon Bedrock models in `ap-southeast-1` (Claude Haiku 4.5 through a global cross-Region inference profile).
- No future information: training and live prediction use the same data-publication rule, and tests check that a forecast for date t does not change when prices after t are edited.
- Safe re-runs: forecasts are written to DynamoDB with conditional writes, so a published record is never overwritten.
- GitHub Actions runs `ruff`, `pytest` (including CDK synth tests) and the web type-check, tests and build on every push and pull request.

### 5. Timeline & Milestones
**Project Timeline**

| Period | Work |
|---|---|
| 12/10 – 25/10 | Data pipeline, baseline, CI, API contract, first model experiments |
| 26/10 – 08/11 | First automatic forecast (27/10), chatbot on AWS (02/11), web on Amplify (04/11), shadow challenger (04/11), internal chatbot demo (08/11) |
| 09/11 – 21/11 | Rolling-accuracy job and API (10–12/11), official chatbot and model evaluations, report chapters |
| 22/11 | Feature freeze |
| 23/11 – 29/11 | Final checks, demo video, report and workshop hand-in |

### 6. Budget Estimation
The budget estimate for the whole system uses the AWS Pricing Calculator table prepared by the team: [CẦN XÁC NHẬN: AWS Pricing Calculator link].

### Infrastructure Costs
- **Budget guardrail:** about 200 USD of AWS credit, AWS Budgets set at 50 USD/month, and Cost Anomaly Detection with email alerts.
- **Cost per service (from the Pricing Calculator):** [CẦN XÁC NHẬN: per-service table from the team's Pricing Calculator estimate]
- **Actual spend so far:** about 5.28 USD from 12/10 to 11/11/2026, covered by credit. The forecast for 12/11–29/11 was about 1.14 USD.
- **Largest variable cost, the chatbot:** about 0.012 USD per question with Claude Haiku 4.5. The quota of 100 questions per day for the whole system caps it at about 1.2 USD per day, and about 36 USD per month in the worst case, which stays under the 50 USD budget.
- **Cost-saving choices:** batch forecasts instead of a real-time endpoint, Lambda outside a VPC (no NAT Gateway), DynamoDB on-demand, 1-month log retention, at most 5 images per ECR repository.

### 7. Risk Assessment
#### Risk Matrix

| Risk | Impact | Likelihood |
|---|---|---|
| Yahoo Finance / yfinance blocks access or its terms forbid our use | High | Medium |
| No model beats the 252-session baseline | Medium | High |
| The chatbot misreads a number or drifts toward investment advice | High | Medium |
| Bedrock cost grows with usage | Medium | Low |
| Manual deployment from one machine breaks a setting (CORS origin, alarm email) | Medium | Medium |
| An evaluation file is overwritten | Medium | Low |
| The daily schedule silently does not run | Medium | Low |

#### Mitigation Strategies
- **Data source:** cite the source on the site, provide no price download, and keep a switch (`VITE_SHOW_METAL_PRICES=false`) that hides metal prices.
- **Models:** keep the baseline in production and only change the published model when a pre-registered criterion is met and an ADR is written.
- **Chatbot:** numbers only come from tools; citations are built on the server; a daily quota per user and for the whole system; and a benchmark with a policy check.
- **Cost:** AWS Budgets, Cost Anomaly Detection and the chat quota.
- **Deployment:** always run `cdk diff` first, deploy one stack at a time, one person at a time.
- **Data safety:** S3 versioning (old versions kept 30 days), conditional writes, explicit deny on deleting evaluation artifacts.
- **Missed runs:** a manual check after every 19:15 run until an "is it running" alarm exists.

#### Contingency Plans
Every risky change can be undone with one deploy command or one web upload, without rebuilding images: turn the shadow model off, go back to an older predict or ingest image, turn the accuracy schedule off, point CORS back to `localhost`, or upload the previous web build. Data is not lost on rollback because the lake bucket is versioned and the forecast table and the user pool are kept (`RETAIN`) when a stack changes.

### 8. Expected Outcomes
#### Technical Improvements:
- Forecasts published automatically every Tuesday to Saturday at 19:15 (Vietnam time), with no manual step.
- A reliability block that is computed only from forecasts that were really published, and that is shown once 20 base dates are available.
- A chatbot whose numbers always match the forecast page, with news sources it can cite.

#### Long-term Value
- A reproducible, low-cost serverless template for a data + ML + GenAI product on AWS.
- Clear next steps: signals beyond price data, switching the published model only through a new ADR, Bedrock Guardrails, a model router to lower chatbot cost, an alarm for missed runs, separate `dev`/`prod` environments with a deployment pipeline, and a price-history chart if the data source allows it.
