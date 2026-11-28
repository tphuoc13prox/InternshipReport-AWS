---
title: "Week 8 Worklog"
date: 2026-11-02
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:
- Automate serverless deployments with AWS CodePipeline.
- Learn Infrastructure as Code with CloudFormation and AWS CDK, and deploy a three-tier architecture.
- Hold team project meetings on the forecast display, data design and price ingestion pipeline.
- Prepare the chatbot demo of 08/11 (script, slides) and write the chatbot chapter of the defense report.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| Mon | **- Practice the lab: Serverless CI/CD with AWS CodePipeline**<br>+ Preparation<br>+ Build the SAM pipeline: create the Git repository and the SAM pipeline<br>+ Test and clean up<br>**- Team project meeting: decide how to display forecasts and design the data**<br>+ The web shows q10/q50/q90 converted to price ranges, the probability of a rise, and switches USD/VND<br>+ Designed the DynamoDB `Forecasts` table (PK `asset`, SK `base_date`)<br>+ Finalized the price sources: GLD, SLV, BZ=F, CL=F, VND=X; cross-checked with EIA<br>**- Team project: chatbot demo script for 08/11**<br>+ Fixed demo script for the chatbot<br>+ Haiku prompt v3 dev results and a check of A-type questions with prompt v2<br>+ Relabel two items in the demo news set according to the labeling-v3 plan | 02/11/2026 | 02/11/2026 | [Serverless - CI/CD with AWS CodePipeline](https://000084.awsfcaj.com/)<br>[Claude parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-claude.html) |
| Tue | **- Practice the lab: AWS CloudFormation**<br>+ Preparation: create an IAM user<br>+ CloudFormation basics: launch AWS CloudShell, create a CloudFormation template<br>+ CloudFormation advanced:<br>* Custom resources with a Lambda function, create the stack, connect to the EC2 instance<br>* Mappings and StackSets<br>* Drift detection<br>+ Clean up resources<br>**- Team review: demo script and chat panel**<br>+ Apply the review on the demo script (PR #77)<br>+ The team connected the web ChatPanel to POST /chat | 03/11/2026 | 03/11/2026 | [AWS CloudFormation](https://000037.awsfcaj.com/)<br>[AWS Amplify Hosting](https://docs.aws.amazon.com/amplify/latest/userguide/welcome.html) |
| Wed | **- Practice the lab: AWS CDK basics**<br>+ CDK basics: create the workspace, set up AWS CloudShell<br>+ Create a CDK template<br>+ Update the CDK template<br>+ Clean up resources<br>**- Team project: demo slides and router rules**<br>+ Marp slides for the chatbot demo<br>+ Lock the router rules before the held-out run | 04/11/2026 | 04/11/2026 | [CDK basic](https://000038.awsfcaj.com/) |
| Thu | **- Team project meeting: build the price ingestion pipeline following the data design**<br>+ The `ingest_prices` CLI writes data to S3 (`raw/`, `clean/prices/`) using the NYSE/CME trading calendars<br>+ Drop the current day's bar by New York time, flag abnormal rows, never forward-fill the target<br>+ Package the Lambda ingest as a container image with a pinned base image digest<br>**- Team project: chatbot chapter for the defense report**<br>+ Write the chatbot chapter and apply the review<br>+ Add the latest EIA weekly report to the demo news | 05/11/2026 | 05/11/2026 | [Lambda container image](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html)<br>[Generative AI Lens](https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/generative-ai-lens.html) |
| Fri | **- Practice the lab: Introduction to Infrastructure as Code**<br>+ Preparation<br>+ Create a Lambda function<br>+ Create the VPC and EC2<br>+ Deploy a three-tier architecture with a CloudFormation stack<br>+ Check access to the web, application and database tiers<br>+ Clean up resources<br>**- Team project: router held-out result and Haiku prompt v4**<br>+ Call the 08/11 session an internal chatbot trial in the demo-news guide<br>+ Record the router held-out result (not passed)<br>+ Haiku prompt v4: plain text, answer template, default dates for get_news (finished over the weekend) | 06/11/2026 | 06/11/2026 | [Introduction to Infrastructure as Code with AWS CloudFormation](https://000102.awsfcaj.com/)<br>[Bedrock prompt engineering](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html) |

### Week 8 Achievements:
1. Built a SAM pipeline with CodePipeline.
2. Wrote CloudFormation templates with custom resources, mappings, StackSets and drift detection.
3. Defined infrastructure in code with AWS CDK and deployed a three-tier architecture with CloudFormation.
4. Prepared the script and slides of the chatbot demo, wrote the chatbot chapter of the defense report and moved to Haiku prompt v4.
5. Agreed with the team on the forecast display, the DynamoDB `Forecasts` table, the price sources and the ingestion pipeline.
