---
title: "Blogs Posted"
date: 2025-09-09
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

5 technical blogs published by our team on the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/), one post by each member.

### [Blog 1 - Prompt Caching for Chatbots with Tool Use: Place cachePoint in the Right Spot to Cut Up to ~90% of the Cost of Repeated Prompt Content on Amazon Bedrock](3.1-Blog1/)

Compiled from the AWS documentation, with illustrative numbers. The post explains how prompt caching works in the Converse API (`cachePoint` in tools, system, and messages; cache write and cache read prices; TTL), estimates the savings for the repeated part of a tool use loop, and proposes how to arrange the prompt so the cache actually hits (static first, dynamic last; a stable tool list; a moving checkpoint at the end of the latest message), along with the pitfalls and how to measure the hit rate.

Published by Phan Tấn Phước on November 17, 2026: [original post on AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2340410270057265/)

### [Blog 2 - The Order Was Placed but the Warehouse Never Got the Event: How Does the Transactional Outbox Solve This on AWS?](3.2-Blog2/)

The post explains the Dual Write Problem in microservices and how the Transactional Outbox Pattern solves it: the order and the outbox event are written in one DynamoDB transaction (`TransactWriteItems`), then DynamoDB Streams, EventBridge Pipes, EventBridge, and SQS deliver the event asynchronously. It also covers why consumers still need idempotency and when this architecture is worth it.

Published by Huỳnh Ngọc Tuấn on November 16, 2026: [original post on AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338993156865643/)

### [Blog 3 - The Model Only Runs a Few Seconds a Day: Do You Really Need a SageMaker Endpoint?](3.3-Blog3/)

The post compares the four SageMaker inference options and shows why a small nightly forecasting job fits a Lambda container image better than a real-time endpoint. It walks through a 7-step pipeline (ECR, EventBridge Scheduler, Lambda destinations, S3, DynamoDB, CloudWatch), the pitfalls found while testing (`awslambdaric`, `WORKDIR`, synchronous vs asynchronous invocation), the cost against the Lambda free tier, and a guide on what to choose when.

Published by Trịnh Tiến Khải on November 15, 2026: [original post on AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338607513570874/)

### [Blog 4 - From SageMaker to the Device: Deploying ML Models to the Edge with AWS IoT Greengrass V2](3.4-Blog4/)

Based on the AWS IoT Greengrass V2 Developer Guide. The post explains why to run inference on the device, the core concepts (core device, component, deployment, nucleus), the model/runtime/inference component split, the requirements for the S3 bucket and the device, how to customize the sample components, and the points to watch out for (Edge Manager discontinued, nucleus versions, pinned runtime versions).

Published by Nguyễn Tiến Long on November 21, 2026: [original post on AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2344828116282147/)

### [Blog 5 - One Job Every Night, Then the Next Step: Do You Need a 24/7 Server or Step Functions?](3.5-Blog5/)

The post shows how to replace a server running cron with EventBridge Scheduler, Lambda (from a container image when libraries are heavy), and Lambda Destinations to chain two steps, with CloudWatch alarms for monitoring. It covers the retry policies of Scheduler and Lambda, a least-privilege IAM role against the confused deputy problem, and when to move to Step Functions.

Published by Châu Trần Mạnh Tiến on November 15, 2026: [original post on AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338839130214379/)
