---
title: "Workshop overview"
date: 2025-09-09
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### The problem

Gold, silver and crude oil prices react quickly to wars, interest-rate decisions and OPEC+ output decisions. In Vietnam, domestic gold and silver follow world prices, and crude oil feeds into retail fuel prices, so many people care about where these prices go next. Today's price is easy to find. What is missing is a forecast that comes with numbers, shows how uncertain it is, and has a track record you can check.

#### What CPAWS does

For each asset and each horizon h = 1–7 trading sessions, the CPAWS website shows:

+ a **forecast range**: the 10th, 50th and 90th percentiles of the return, converted into a price range in USD or VND;
+ the **probability of going up**;
+ a **reliability block** that scores the forecasts the system actually published, once enough of them have matured.

A Vietnamese **chatbot** on Amazon Bedrock explains the forecasts and lists related Fed/EIA news with links. It takes every number from the system through tools and declines to give investment advice.

| Asset | Series used for the forecast | Note |
|---|---|---|
| Gold | GLD (ETF holding physical gold) | Proxy series; gold prices on the site are converted from GLD, not spot prices |
| Silver | SLV (ETF holding physical silver) | Proxy series; silver prices are converted from SLV |
| Brent | BZ=F (front-month futures) | Jumps on contract roll dates |
| WTI | CL=F (front-month futures) | Jumps on contract roll dates |

#### Why forecast returns instead of prices

The system forecasts the distribution of the **log return** h sessions ahead as seven quantiles (q05 to q95), not the price itself. Tomorrow's price is usually very close to today's, so a "model" that just repeats today's price already looks almost perfect on price-level error. Forecasting returns removes that illusion: a model only scores well if it learns something about the change. The web app then converts the quantiles back into prices.

The range from q10 to q90 is called the **nominal 80% forecast range**. How often real prices actually land inside it is measured separately (`coverage_80`) and can differ from 80%.

#### What runs in production today

The published forecast for all four assets is `baseline-q252`, a **252-session statistical baseline**: for each h it takes the empirical quantiles of the completed h-session returns of the last 252 sessions. It has no learned parameters. The team tried 13 configurations of other models and none beat this baseline under the criterion fixed before each experiment (details in [5.11](../5.11-lessons-learned/)). A challenger called S1a runs next to it in shadow mode; users do not see it.

#### The team

| Member | Role | Main responsibilities |
|---|---|---|
| Trịnh Tiến Khải | Research Lead | Models, data, the predict Lambda, report coordination |
| Nguyễn Tiến Long | Modeling & Evaluation | Evaluation, champion/challenger |
| Phan Tấn Phước | News & Chatbot | News labelling, chatbot |
| Châu Trần Mạnh Tiến | Platform | CDK, Lambda, web, deployment |
| Huỳnh Ngọc Tuấn | Data & Security Lead | IAM, API Gateway, Cognito, DynamoDB, cost |

The team's source code is public; the link is in [5.2](../5.2-prerequiste/).

#### What you will build

By the end of the workshop you will have the same five CDK stacks running in your own account, in `ap-southeast-1`:

+ a schedule that runs ingest → predict every Tuesday to Saturday at 19:15 Vietnam time;
+ four DynamoDB tables and an S3 data lake;
+ a Cognito-protected REST API with `GET /forecasts/{asset}`, `GET /accuracy/{asset}` and `POST /chat`;
+ a web app on Amplify Hosting;
+ six CloudWatch alarms that email you through SNS.

Sign-up for the web app: for the grading period, self sign-up with email verification is open (ADR-0014, deployed on 22/11/2026) and will be closed again after grading. Details in [5.5.1](../5.5-api-and-chatbot/5.5.1-cognito-and-api/).
