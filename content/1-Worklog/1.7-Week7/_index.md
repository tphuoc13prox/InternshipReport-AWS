---
title: "Week 7 Worklog"
date: 2026-10-26
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:
- Build serverless back-ends with Lambda, S3, DynamoDB, API Gateway and AWS SAM.
- Monitor and trace serverless applications with CloudWatch and X-Ray.
- Write and get approval for the POST /chat contract, and build the chatbot Lambda with Bedrock.
- Review the project architecture against AWS Well-Architected with the team.
- Prepare a hand-labeled news set for the chatbot's get_news tool.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| 2 | **- Team meeting: review the Well-Architected criteria and revise the project architecture**<br>+ Batch forecasting with a Lambda container (NumPy) instead of an always-on SageMaker endpoint<br>**- Team project: benchmark set v2 and get_forecast v2**<br>+ Question set v2 with an interval-label check and SHA-256 lock (prepared over the weekend)<br>+ get_forecast v2 shares the formatter of the forecast API Lambda<br>+ Revise prompt v2 after review and dev runs<br>**- Attend Event 4: Smart Plushies for Children - Amazon volunteering (afternoon)**<br>+ Prepare smart plush toys for children supported by Operation Smile Vietnam | 26/10/2026 | 26/10/2026 | [AWS Well-Architected](https://aws.amazon.com/architecture/well-architected/)<br>[Bedrock tool use](https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use.html) |
| 3 | **- Practice the lab: Serverless - Front-end calling API Gateway**<br>+ Introduction to API Gateway and DynamoDB<br>+ Deploy the front-end<br>+ Create the DynamoDB table and Lambda functions that write, read and delete data<br>+ Set up API Gateway: create methods, configure and enable CORS<br>+ Test the API with Postman and with the front-end<br>+ Repeat the flow with the updated version of the lab (000135)<br>+ Clean up resources<br>**- Team project: POST /chat contract and chatbot Lambda**<br>+ Propose the POST /chat contract, apply the reviews and get it approved<br>+ Chatbot Lambda core: Claude Haiku on Bedrock with the get_news and get_forecast tools<br>+ Deadline from requestTimeEpoch, bounded Bedrock calls, daily quota in DynamoDB<br>+ Collect 69 Fed/EIA candidate releases for labeling | 27/10/2026 | 27/10/2026 | [Serverless - Build Frontend to call API Gateway](https://000079.awsfcaj.com/)<br>[Serverless - A Guide to Writing Front-End Code for Calling API Gateway (updated version)](https://000135.awsfcaj.com/) |
| 4 | **- Practice the lab: Serverless with Lambda, API Gateway and AWS SAM**<br>+ Preparation: Cloud9 instance, CodeCommit repository<br>+ Deploy the sample app: front-end, back-end infrastructure, trip data, configuration check<br>+ Deploy the ride-times system (back-end and front-end)<br>+ On-ride photo processing: chroma-key and photo-compositing Lambda functions, post-processing<br>+ Test the features and clean up<br>**- Team project: news labeling v3**<br>+ Draft the labeling-v3 prompt, label template and case builder<br>+ Hand-label 50 Fed/EIA releases and 3 special cases<br>+ Fix the interval-label check of the benchmark | 28/10/2026 | 28/10/2026 | [Serverless with Lambda, API Gateway and SAM](https://000066.awsfcaj.com/)<br>[Bedrock prompt engineering](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html) |
| 5 | **- Practice the lab: Monitoring serverless applications with CloudWatch and X-Ray**<br>+ Preparation<br>+ Monitor with CloudWatch: debug with CloudWatch Logs, create custom metrics, create CloudWatch alarms<br>+ Trace requests with AWS X-Ray<br>+ Clean up resources<br>**- Finish the whole CPAWS proposal**<br>+ My part was news and the chatbot, supporting the proposal: the hand-labelled Fed/EIA news set for get_news and the POST /chat Lambda<br>**- Team project: demo news set for get_news**<br>+ Hand-labeled Fed/EIA news set and its guide<br>+ Labeling-v3 real run results and manual evidence sheet<br>+ Error handling: skip invalid news, fixed answer at the tool-round limit, 504 on model timeout, 400 on bad base64 | 29/10/2026 | 29/10/2026 | [Serverless - Monitoring Serverless app with CloudWatch and X-Ray](https://000085.awsfcaj.com/)<br>[Converse API reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html)<br>[Proposal Sample](https://workshop-sample.awsfcaj.com/2-proposal/) |
| 6 | **- Practice the lab: Serverless - Lambda with S3 and DynamoDB**<br>+ Image processing on AWS: create the image-resizing Lambda function, the S3 bucket and the IAM policy, test the function<br>+ Write data to Amazon DynamoDB: create and manage tables, create a Lambda function that writes data<br>+ Clean up resources<br>**- Team project: ADR-0013 and chatbot benchmark v2 results**<br>+ ADR-0013: use manual news labels until the defense, no Bedrock labeling in production<br>+ Official results of chatbot benchmark v2 and the decisions recorded with the team<br>+ Draft a rule-based router for the chatbot<br>+ Merge the chat API branch (PR #55); the team deployed the POST /chat Lambda the same day<br>+ Over the weekend: Haiku prompt v3 and dev-only benchmark sets | 30/10/2026 | 30/10/2026 | [Serverless - Interaction between Lambda with S3 and DynamoDB](https://000078.awsfcaj.com/)<br>[Bedrock prompt routing](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-routing.html)<br>[Bedrock Guardrails](https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html) |

### Week 7 Achievements:
1. Built serverless flows: image resizing with Lambda and S3, DynamoDB writes, a front-end calling API Gateway with CORS, and a full AWS SAM application.
2. Debugged Lambda with CloudWatch Logs, custom metrics and alarms, and traced requests with X-Ray.
3. Got the POST /chat contract approved and built the chatbot Lambda (Claude Haiku on Bedrock, get_news/get_forecast tools, timeouts, DynamoDB quota).
4. Hand-labeled Fed/EIA releases for get_news and recorded the manual-label decision in ADR-0013.
5. Reviewed the architecture against AWS Well-Architected with the team and moved to batch forecasting with a Lambda container.
6. Took part in the Amazon volunteering activity (Event 4).
7. Finished the whole CPAWS proposal with the team; my part was news and the chatbot (hand-labelled news set, get_news, POST /chat).
