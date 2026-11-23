---
title: "Blog 5"
date: 2026-11-15
weight: 5
chapter: false
pre: " <b> 3.5. </b> "
---

# One Job Every Night, Then the Next Step: Do You Need a 24/7 Server or Step Functions?

by Châu Trần Mạnh Tiến | November 15, 2026 | published in [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Original post on Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338839130214379/)

*A post by team member Châu Trần Mạnh Tiến. The post was written and published in Vietnamese; this page is an English translation that stays faithful to the published version, with the full links of the references.*

There is a very common kind of problem: every day at a fixed time, call an external API to fetch data, then process that data and store the results. The most familiar approach is to rent a server and set up cron. But if each job only runs for a few dozen seconds a day, keeping a machine running 24 hours is quite wasteful, and you also have to patch and monitor that machine yourself.

When I studied and tried building this flow the serverless way on AWS, I found that combining a few services is enough: Amazon EventBridge Scheduler, AWS Lambda, and Lambda Destinations. In this post I summarize how to put them together, along with the easy mistakes I ran into.

### Overview diagram

```text
EventBridge Scheduler (fixed time every day)
        │ asynchronous invocation
        ▼
"Fetch data" Lambda ──► stores in S3
        │ Lambda Destinations (OnSuccess / OnFailure)
        ▼
"Process" Lambda ──► writes results to DynamoDB

CloudWatch Alarm (Errors metric) ──► SNS ──► email
```

![Architecture of EventBridge Scheduler, Lambda, and Lambda Destinations](/images/3-Blog/blog5_figure1.jpg)

*The image attached to the original post: the architecture diagram of the flow.*

### 1. EventBridge Scheduler: scheduling instead of cron

EventBridge Scheduler lets you create cron-based or rate-based schedules, choose a time zone for each schedule, create one-time schedules, and use a flexible time window when the job does not have to run at an exact minute.

The point I think deserves attention is the retry policy. According to the AWS documentation, the maximum number of retries can be set from 0 to 185, with a default of 185, and the maximum event age for retries can be from 60 seconds to 24 hours. Events that fail to be delivered can be sent to an SQS queue used as a dead-letter queue (DLQ).

Scheduler needs an IAM role to invoke Lambda. This role should only have the `lambda:InvokeFunction` permission on exactly one function. The trust policy should add the `aws:SourceAccount` and `aws:SourceArn` conditions so that only schedules in the right account and the right schedule group can use the role. This is the approach AWS recommends to avoid the "confused deputy" problem.

### 2. Lambda from a container image when the libraries are too heavy

If the code needs large libraries such as pandas or machine learning libraries, a Lambda zip package easily exceeds the 250 MB unzipped limit. In that case you can package it as a container image (up to 10 GB), push it to Amazon ECR, and have Lambda run directly from the image.

In exchange, a large image means slower cold starts. On a first run I once saw the Lambda init phase take more than 10 seconds because it had to import heavy libraries. Lambda retried the init on its own, so the run still succeeded, but you should watch the `INIT_REPORT` and `REPORT` lines in CloudWatch Logs.

Every rebuild adds another image to ECR. Set a lifecycle rule (for example, keep the 5 newest images) so you do not pay to store old images.

### 3. Lambda Destinations: chaining two steps without Step Functions

For asynchronous invocations, Lambda lets you configure where the result goes when a run succeeds (OnSuccess) and when it fails after all retries are used up (OnFailure). The destination can be another Lambda function, an SQS queue, an SNS topic, an EventBridge event bus, and, for OnFailure, also an S3 bucket.

According to the documentation, for asynchronous invocations Lambda retries twice by default if the function returns an error (configurable from 0 to 2), and keeps the event in the queue for up to 6 hours.

For a flow with only two steps, pointing OnSuccess to the "process" Lambda is enough. If the first step partially fails but still produces usable data, you can also point OnFailure to the second step.

An example with the AWS CDK (Python):

```python
fetch_fn = lambda_.DockerImageFunction(
    self, "FetchFunction",
    code=lambda_.DockerImageCode.from_ecr(repo, tag_or_digest="v1"),
    memory_size=1024,
    timeout=Duration.seconds(300),
    retry_attempts=1,
    on_success=destinations.LambdaDestination(process_fn),
    on_failure=destinations.LambdaDestination(process_fn),
)
```

### 4. Monitoring: know right away when something fails

An automated flow without alerts can easily break silently. A simple approach is to create a CloudWatch Alarm on the Errors metric of each Lambda function, with a threshold of 1 error or more, and send it through SNS to email.

### Things to keep in mind

- **Destinations only run for asynchronous invocations.** Scheduler invokes asynchronously, so the flow works. But when you test it yourself with the default `aws lambda invoke` (synchronous), the second step is not called. To test the whole flow, add `--invocation-type Event`.
- **Keep retries at only one layer.** Scheduler retries many times by default, and Lambda has its own retries too. I set the Scheduler retries to 0 and kept the retries in Lambda, which makes it easier to predict how many times the external API is called and easier to read the logs.
- **Lambda runs for at most 15 minutes per invocation.** Longer jobs should use ECS, AWS Batch, or Step Functions to split them up.
- **When to move to Step Functions:** when the flow has many steps, branches, waits, parallel runs, or when you need to see the status of each step on one screen. Destinations suit short, simple chains.
- **If you use the CDK, always run `cdk diff` before `cdk deploy`.** Read the `[-]` lines (resources about to be deleted) carefully before confirming.

### Conclusion

For jobs like "run once a day for a few dozen seconds, then do one more step", the combination of EventBridge Scheduler + Lambda + Lambda Destinations is enough, without a server running all day or a state machine. You only pay for the actual run time, and you get alerts when something fails.

This is what I learned and tried myself while studying, and there are surely gaps. I would welcome feedback, especially from those who have run this kind of flow in production for a long time.

### References

- [Amazon EventBridge Scheduler – User Guide](https://docs.aws.amazon.com/scheduler/latest/UserGuide/what-is-scheduler.html)
- [RetryPolicy – EventBridge Scheduler API Reference](https://docs.aws.amazon.com/scheduler/latest/APIReference/API_RetryPolicy.html)
- [Configuring error handling for Lambda asynchronous invocations](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async-error-handling.html)
- [Lambda Destinations for asynchronous invocations](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async-retain-records.html)
- [Create a Lambda function using a container image](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html)
- [The confused deputy problem – IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html)

#AWS #AWSStudyGroupVN #FCAJ #Serverless #AWSLambda #EventBridge #AWSCDK
