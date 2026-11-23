---
title: "Blog 3"
date: 2026-11-15
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# The Model Only Runs a Few Seconds a Day: Do You Really Need a SageMaker Endpoint?

by Trịnh Tiến Khải | November 15, 2026 | published in [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Original post on Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338607513570874/)

*A post by team member Trịnh Tiến Khải. The post was written and published in Vietnamese; this page is an English translation that stays faithful to the published version, with the full links of the references.*

Recently I have been looking into the ways to run inference for machine learning models on AWS, and I found something quite interesting: for many problems, the most important question is not "which model" but "how is the model called". In this post I write down what I read, with an example diagram that numbers each flow (attached image) so it is easier to follow.

**The scenario:** an online store wants to forecast tomorrow's number of orders for each warehouse. It runs once a night after the day's data is closed; the model runs for a few seconds and the results are written to a database for a dashboard to read. Nobody needs the model to answer on every click.

The first instinct is usually "it's a machine learning model, so put it on a SageMaker endpoint". But after reading the pricing carefully, I found that this choice does not fit this kind of workload.

### 1. Four ways to run inference on SageMaker

- **Real-time endpoint:** billed per instance-hour for as long as the endpoint is running, even when there are no requests. A good fit when you need low latency, steady traffic, or a GPU.
- **Serverless Inference:** scales to zero when idle, at the cost of cold starts. Memory can be set from 1 GB to 6 GB, and there is no GPU.
- **Asynchronous Inference:** requests are put in a queue; a good fit for large payloads or long processing times.
- **Batch Transform:** runs in batches over a whole dataset and shuts down when done.

For a job of a few seconds a day, a real-time endpoint is basically paying for a machine that sits and waits. Batch Transform or Serverless Inference would work, but if the model is small and can run on a CPU, there is an even simpler option: AWS Lambda.

### 2. Why a Lambda container image instead of a zip package

- A Lambda zip package is limited to 250 MB unzipped (including layers). Data libraries such as pandas, pyarrow, and scikit-learn together easily hit that limit.
- A container image can be up to 10 GB (uncompressed, including all layers).
- The maximum timeout is still 15 minutes. Moving to a container does not change this limit.

What I like most: the same image runs on a local machine for testing before it is pushed to Amazon ECR, so "it runs on my machine" and "it runs on Lambda" are the same thing.

### 3. Following the 7 flows in the diagram

1. Build the image, pin the base image by digest, tag it with the commit SHA, and push it to Amazon ECR.
2. EventBridge Scheduler runs on a nightly cron schedule and invokes the prepare Lambda asynchronously.
3. The prepare Lambda collects and cleans the data and writes it to S3 by date (`data/dt=…/`).
4. When prepare finishes, a Lambda destination invokes the inference Lambda. You can wire both OnSuccess and OnFailure: if one data source fails, inference still runs on the data that is still good.
5. The inference Lambda reads the data and the model weights in `models/`.
6. The results are written to DynamoDB for the dashboard or an API to read.
7. Logs and metrics go to CloudWatch: one summary log line per run and one alarm on the Errors metric are enough to know the next morning how last night's run went.

![A nightly batch inference pipeline with a Lambda container image](/images/3-Blog/blog3_figure1.jpg)

*Image: an example diagram of a nightly batch inference pipeline, which I drew in draw.io based on the AWS documentation.*

The red box in the bottom corner of the diagram is the option of a real-time endpoint running 24/7, which does not fit this kind of workload.

### 4. What I discovered while reading the documentation

**A small model does not have to bring a whole framework to Lambda.** For linear models or small networks, you can train elsewhere, export the weights to a file (for example `.npz`), and at prediction time only need NumPy to multiply matrices. The image is much lighter and starts faster.

**With a non-AWS base image, you must install the runtime interface client yourself.** The AWS base image (`public.ecr.aws/lambda/python`) already includes it. If you use another image such as `python:3.12-slim`, you must install `awslambdaric`; without it, Lambda cannot call the handler.

```dockerfile
# Simplified illustration
FROM python:3.12-slim@sha256:<digest>
WORKDIR /function
COPY requirements.txt .
RUN pip install --no-cache-dir awslambdaric -r requirements.txt
COPY src/ ./src/
ENTRYPOINT ["python", "-m", "awslambdaric"]
CMD ["src.inference.handler"]
```

One more detail I found while testing: `awslambdaric` looks for the handler module relative to the container's working directory. If the working directory is not where `src/` lives, Lambda reports `Runtime.ImportModuleError`. So it is better to declare `WORKDIR` explicitly instead of relying on the base image's default.

**With asynchronous invocation, Lambda retries on its own.** By default, Lambda retries twice when the function returns an error. You can set the number of retries from 0 to 2, and the maximum age of an event in the queue from 60 seconds to 6 hours. For an inference job, setting retries to 0 is usually more sensible: if it fails, it stops, the alarm fires, and an operator reads the logs and reruns it in a controlled way.

**For direct invocations of a function, Lambda destinations apply only to asynchronous invocations** (event source mappings for Kinesis, DynamoDB Streams, and Kafka also have their own OnFailure destination). The destination can be another Lambda, SQS, SNS, EventBridge, or S3 (S3 only for OnFailure). An easy trap: `aws lambda invoke` is synchronous by default (`RequestResponse`), so when you run it by hand the first function finishes but the destination does not fire. To rerun the whole chain you must call:

```bash
aws lambda invoke --function-name <prepare> --invocation-type Event out.json
```

### 5. Cost

Lambda has a free tier of 1 million requests and 400,000 GB-seconds per month, which does not expire after 12 months like many other services (shared across the whole account). For example, a 1 GB function that we assume always runs the full 120 seconds, once a night, uses only about 3,600 GB-seconds a month, less than 1% of the free tier. ECR charges for image storage by size, so set a lifecycle policy to keep only the last few images.

### What to choose when

- Sparse batch runs, a small model, CPU only, under 15 minutes: **Lambda container image**.
- Irregular traffic, cold starts acceptable, no GPU needed: **SageMaker Serverless Inference**.
- Large payloads or long processing: **SageMaker Asynchronous Inference**.
- Scoring a whole large dataset in batches: **SageMaker Batch Transform**.
- Continuous millisecond latency, a large model, or a GPU: **SageMaker real-time endpoint**.

### Conclusion

Choose the service by how the workload actually runs, not by the label "this is an ML problem". For a forecasting job that runs a few seconds every night, a Lambda container image has enough room for the libraries, runs exactly the same locally, and costs almost nothing. The rest is understanding the invocation type (synchronous or asynchronous), because many Lambda features such as destinations and retries apply to only one of them.

*Note: prices and service limits may change, so check the AWS Pricing pages and the latest documentation before applying this. The Dockerfile is a simplified illustration.*

### References

- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [Create a Lambda function using a container image](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html)
- [Lambda asynchronous invocation](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html)
- [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/)
- [SageMaker Serverless Inference](https://docs.aws.amazon.com/sagemaker/latest/dg/serverless-endpoints.html)
- [Inference options in Amazon SageMaker AI](https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model-options.html)

#AWS #AWSLambda #AmazonECR #AmazonSageMaker #Serverless #MLOps #AWSStudyGroup #FCAJ
