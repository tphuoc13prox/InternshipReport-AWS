---
title: "Workshop"
date: 2026-11-20
weight: 5
chapter: false
pre: " <b> 5. </b> "
---
# CPAWS - Building a Serverless Commodity Price Forecasting Platform on AWS

#### Overview

**CPAWS** is a web application that forecasts gold, silver, Brent crude and WTI crude for the next 1–7 trading sessions, shown in USD and VND, with a Vietnamese chatbot on Amazon Bedrock that explains the forecasts. The system runs serverless on a schedule: no endpoint, server or NAT Gateway runs 24/7.

This workshop describes the system that the CPAWS team (5 members, FCAJ program) actually runs in the team's AWS account in region `ap-southeast-1`. Every command, configuration and measurement comes from the source code and from real runs. Anything not done, or done without meeting its target, is stated as such.

In this workshop, you will learn how to:
- Collect prices by trading calendar, clean them and store them in a data lake on **Amazon S3**
- Build a quantile forecasting baseline without look-ahead leakage and evaluate it with pinball loss
- Deploy infrastructure with **AWS CDK**: S3, ECR, **AWS Lambda** container images, **Amazon DynamoDB**
- Automate the daily flow with **Amazon EventBridge Scheduler** and Lambda Destinations
- Expose an authenticated API with **Amazon API Gateway** and **Amazon Cognito**, and host the web app on **AWS Amplify Hosting**
- Call the **Amazon Bedrock** Converse API with tool use so the chatbot only uses system data
- Test, monitor with **Amazon CloudWatch**, estimate and track cost, and review the design against AWS Well-Architected

#### Contents

1. [Introduction](5.1-workshop-overview/)
2. [Prerequisites](5.2-prerequiste/)
3. [Data and model](5.3-data-and-model/)
4. [Infrastructure with CDK](5.4-infrastructure/)
5. [Chatbot with Amazon Bedrock](5.5-chatbot/)
6. [Testing and monitoring](5.6-testing/)
7. [Cost and Well-Architected](5.7-cost/)
8. [Cleanup](5.8-cleanup/)

#### Source code

Team repository: [CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting](https://github.com/CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting) (branch `developer`).
