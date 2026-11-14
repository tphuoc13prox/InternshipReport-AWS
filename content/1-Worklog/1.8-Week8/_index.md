---
title: "Week 8 Worklog"
date: 2026-11-02
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### OBJECTIVES IN WEEK 8:
- Automate serverless deployments with AWS CodePipeline.
- Learn Infrastructure as Code with CloudFormation and AWS CDK, and deploy a three-tier architecture.
- Prepare the chatbot demo of 08/11 (script, slides) and write the chatbot chapter of the defense report.

### TASKS OF WEEK 8:

| Day | Task | Start Date | End Date | References |
|:---:|---|:---:|:---:|---|
| **Mon** | **- Practice the lab: Serverless CI/CD with AWS CodePipeline**<br>+ Preparation<br>+ Build the SAM pipeline: create the Git repository and the SAM pipeline<br>+ Build the front-end pipeline: create the Git repository and the pipeline<br>+ Test the web application and clean up<br>**- Team project: chatbot demo script for 08/11**<br>+ Fixed demo script for the chatbot<br>+ Haiku prompt v3 dev results and a check of A-type questions with prompt v2<br>+ Relabel two items in the demo news set according to the labeling-v3 plan | 02/11/2026 | 02/11/2026 | [Serverless - CI/CD with AWS CodePipeline](https://000084.awsfcaj.com/)<br>[Claude parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-claude.html) |
| **Tue** | **- Practice the lab: AWS CloudFormation**<br>+ Preparation: create an IAM user<br>+ CloudFormation basics: launch AWS CloudShell, create a CloudFormation template<br>+ CloudFormation advanced:<br>* Custom resources with a Lambda function, create the stack, connect to the EC2 instance<br>* Mappings and StackSets<br>* Drift detection<br>+ Clean up resources<br>**- Team review: demo script and chat panel**<br>+ Apply the review on the demo script (PR #77)<br>+ The team connected the web ChatPanel to POST /chat | 03/11/2026 | 03/11/2026 | [AWS CloudFormation](https://000037.awsfcaj.com/)<br>[AWS Amplify Hosting](https://docs.aws.amazon.com/amplify/latest/userguide/welcome.html) |
| **Wed** | **- Practice the lab: AWS CDK basics**<br>+ CDK basics: create the workspace, set up AWS CloudShell<br>+ Create a CDK template<br>+ Update the CDK template<br>+ Clean up resources<br>**- Team project: demo slides and router rules**<br>+ Marp slides for the chatbot demo<br>+ Lock the router rules before the held-out run | 04/11/2026 | 04/11/2026 | [CDK basic](https://000038.awsfcaj.com/) |
| **Thu** | **- Practice the lab: AWS CDK basics - part 2**<br>+ Preparation: IAM role, EC2 server, VS Code environment<br>+ Build ECS, Application Load Balancer and API Gateway with CDK<br>+ Build Lambda and S3 with CDK<br>+ Create nested stacks with CDK<br>+ Clean up resources<br>**- Team project: chatbot chapter for the defense report**<br>+ Write the chatbot chapter and apply the review<br>+ Add the latest EIA weekly report to the demo news | 05/11/2026 | 05/11/2026 | [CDK Basic - 2](https://000076.awsfcaj.com/)<br>[Generative AI Lens](https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/generative-ai-lens.html) |
| **Fri** | **- Practice the lab: Introduction to Infrastructure as Code**<br>+ Preparation<br>+ Create a Lambda function<br>+ Create the VPC and EC2<br>+ Deploy a three-tier architecture with a CloudFormation stack<br>+ Check access to the web, application and database tiers<br>+ Clean up resources<br>**- Team project: router held-out result and Haiku prompt v4**<br>+ Call the 08/11 session an internal chatbot trial in the demo-news guide<br>+ Record the router held-out result (not passed)<br>+ Haiku prompt v4: plain text, answer template, default dates for get_news (finished over the weekend) | 06/11/2026 | 06/11/2026 | [Introduction to Infrastructure as Code with AWS CloudFormation](https://000102.awsfcaj.com/)<br>[Bedrock prompt engineering](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html) |

### ACHIEVEMENTS IN WEEK 8:
1. Built a SAM pipeline and a front-end pipeline with CodePipeline.
2. Wrote CloudFormation templates with custom resources, mappings, StackSets and drift detection.
3. Defined infrastructure in code with AWS CDK (ECS, ALB, API Gateway, Lambda, S3, nested stacks) and deployed a three-tier architecture.
4. Prepared the script and slides of the chatbot demo, wrote the chatbot chapter of the defense report and moved to Haiku prompt v4.
