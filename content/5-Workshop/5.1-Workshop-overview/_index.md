---
title: "Introduction"
date: 2026-11-20
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---
#### The problem

Gold, silver and crude oil prices move sharply around major events. Ordinary users face three difficulties:

- **Hard to know the likely range of upcoming prices:** commentary in the press is usually qualitative, with no numbers and no published track record.
- **Hard to understand why prices move:** news is plentiful and scattered.
- **Hard to relate to Vietnamese dong:** world prices are quoted in USD per ounce or USD per barrel.

#### The solution

CPAWS does not forecast a single price. It forecasts the **distribution of the h-session log return** (h = 1–7) as 7 quantiles (0.05 to 0.95), then converts it into:

- an **80% range** (10th to 90th percentile) and the median, shown in USD and VND;
- the **probability of a rise** at each horizon;
- **reliability measured on live data** (`rolling_8w`): how often the realized price fell inside the 80% range over the last 8 weeks.

A Vietnamese chatbot answers questions about these forecasts. Every number is read from the database by a tool, every news item is cited, and the chatbot gives no investment advice.

The model currently published (`baseline-q252`) is a **252-session statistical baseline**: the empirical quantiles of returns over roughly the last trading year. The team tried machine learning models (DLinear, CEEMDAN, volatility-scaled models), but none beat this baseline with enough confidence under the criterion fixed in advance (section [5.3.2](../5.3-data-and-model/5.3.2-baseline-forecast/)). The web app and the chatbot therefore always label the source as "252-session statistical baseline".

#### Current architecture

![CPAWS current architecture](/images/5-Workshop/CPAWS-architecture-current.png)

*Diagram drawn by the CPAWS team in draw.io (layout revised by Khải on 17/11/2026), state as of 13/11/2026. Every component on the diagram is deployed and running.*

The system has three flows:

1. **Daily ingest and forecast** (19:15 Vietnam time, Tuesday–Saturday). EventBridge Scheduler invokes the `ingest` Lambda, which downloads prices and writes a snapshot to S3. When ingest finishes, whether it succeeds or fails, Lambda Destinations invokes the `predict` Lambda. Predict computes the forecast for each asset and writes it to DynamoDB `cpaws-dev-forecasts`. Each published record is written exactly once and never overwritten.
2. **Reliability measurement** (20:45 Vietnam time). The `accuracy-job` Lambda compares past forecasts with the realized prices now known, then writes the results to table `cpaws-dev-accuracy`.
3. **User-facing services.** A React web app on Amplify Hosting signs in with Cognito and calls the REST API (API Gateway + Cognito authorizer) to get forecasts (`/forecasts`), reliability (`/accuracy`) and to ask the chatbot (`/chat`). The chat Lambda calls Claude Haiku 4.5 on Bedrock with two tools, `get_forecast` and `get_news`.

Operations run alongside these flows:
- CloudWatch Logs, metric filters and 6 alarms that send email through Amazon SNS;
- the EIA API key stored in SSM Parameter Store;
- AWS Budgets to track cost;
- infrastructure written in CDK, split into 5 stacks.

#### CDK stacks

| Stack | Main resources |
|---|---|
| `CpawsData` | S3 `cpaws-dev-lake`, ECR `cpaws-dev-ingest` and `cpaws-dev-predict`, Lambda `ingest`, `predict`, `accuracy-job`, two EventBridge Scheduler schedules |
| `CpawsServing` | DynamoDB `forecasts`, `news-labels`, `chat-usage`, `accuracy` (on-demand) |
| `CpawsAuth` | Cognito User Pool, self sign-up disabled |
| `CpawsApi` | REST API stage `dev`; Lambda `forecasts-api`, `accuracy-api`, `chat-api` |
| `CpawsOps` | SNS alarm topic, 6 CloudWatch alarms |

Amplify Hosting, the SSM parameter holding the API key and AWS Budgets are created outside CDK (see [5.4](../5.4-infrastructure/)).

#### Compared with the proposal

During the project the architecture was simplified relative to the [proposal](../../2-proposal/). Each change has an ADR in the repository.

| Proposal | Current state | Reason |
|---|---|---|
| Step Functions and SageMaker AI for weekly retraining | Two Lambdas chained by a destination, no retraining | The 252-session baseline needs no model artifact; ML models did not beat it (ADR-0011, ADR-0012) |
| Secrets Manager | SSM Parameter Store `SecureString` | Only one key, no automatic rotation needed; cost close to zero |
| News labelling with Bedrock (Nova 2 Lite) | 2–3 Fed/EIA releases labelled by hand each week | The check on 49 real releases missed its thresholds (ADR-0013) |
| Router choosing between Nova and Haiku | Every question goes to Claude Haiku 4.5 | The router missed its threshold on a held-out question set (section [5.5](../5.5-chatbot/)) |
| Bedrock Guardrails, Knowledge Bases | Not enabled | Controlled with prompt rules, tools and server-side checks (decision of 11/11/2026) |

#### Product demo

- Demo video: [LINK TO BE ADDED]
- Production (Amplify Hosting): [LINK TO BE ADDED]. The site requires sign-in; a trial account for graders is sent with the submission and is not written on this page.
