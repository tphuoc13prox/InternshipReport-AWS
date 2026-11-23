---
title: "Workshop"
date: 2025-09-09
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# CPAWS – Commodity Price Forecasting on AWS

#### Overview

This workshop walks through **CPAWS**, the serverless system our team built and runs on AWS. Every trading day it collects prices for gold, silver, Brent and WTI, publishes a 1–7 session forecast range for each of them, measures how reliable the published forecasts turned out to be, and serves everything to a web app with a Vietnamese chatbot on Amazon Bedrock.

You will deploy the same building blocks the team uses, in the same order: storage and tables, the daily ingest → predict pipeline, the rolling-accuracy job, sign-in and the REST API, the chatbot, the web app on AWS Amplify, and finally alarms, tests, cost and clean-up. All infrastructure is defined with AWS CDK (Python) in five stacks: `CpawsServing`, `CpawsAuth`, `CpawsData`, `CpawsApi` and `CpawsOps`.

+ **Data pipeline:** Amazon EventBridge Scheduler, AWS Lambda (container images in Amazon ECR), Amazon S3, Amazon DynamoDB.
+ **User-facing services:** Amazon Cognito, Amazon API Gateway, AWS Lambda, Amazon Bedrock, AWS Amplify Hosting.
+ **Operations:** Amazon CloudWatch, Amazon SNS, AWS Systems Manager Parameter Store, AWS Budgets.

{{% notice note %}}
CPAWS is a team project of five members. This write-up covers the whole system; when a part was built by a specific teammate, the page says so. The forecasts are for reference only and are not investment advice.
{{% /notice %}}

#### Content

1. [Workshop overview](5.1-workshop-overview/)
2. [Prerequisite](5.2-prerequiste/)
3. [Architecture and data flow](5.3-architecture/)
4. [Data pipeline](5.4-data-pipeline/)
5. [API, sign-in and chatbot](5.5-api-and-chatbot/)
6. [Web app on AWS Amplify](5.6-web-amplify/)
7. [Monitoring and testing](5.7-monitoring-and-testing/)
8. [Cost estimate and optimization](5.8-cost/)
9. [Demo](5.9-demo/)
10. [Clean up](5.10-cleanup/)
11. [Challenges and lessons learned](5.11-lessons-learned/)
