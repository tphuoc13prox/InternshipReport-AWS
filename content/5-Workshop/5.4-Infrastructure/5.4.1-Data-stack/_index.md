---
title: "Bootstrap and data stack"
date: 2026-11-20
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---
#### Bootstrap CDK

Run once per account and region:

```bash
cd infra
pip install -r requirements.txt
cdk bootstrap aws://<ACCOUNT_ID>/ap-southeast-1 --profile cpaws-dev
```

#### What the `CpawsData` stack contains

| Resource | Configuration |
|---|---|
| S3 `cpaws-dev-lake` | Public access blocked, SSL enforced, SSE-S3 encryption, versioning on, old versions kept 30 days, `RemovalPolicy.RETAIN` |
| ECR `cpaws-dev-ingest`, `cpaws-dev-predict` | Scan on push, keep only the 5 newest images |
| Lambda `cpaws-dev-ingest` | x86_64 container, 1,024 MB, 300-second timeout, 1 asynchronous retry |
| Lambda `cpaws-dev-predict` | x86_64 container, 1,024 MB, 120-second timeout, 0 retries |
| Lambda `cpaws-dev-accuracy-job` | Uses the predict image with its own handler, 1,024 MB, 300 seconds, 0 retries |
| EventBridge Scheduler | Ingest schedule at 12:15 UTC and accuracy job schedule at 13:45 UTC, Tuesday–Saturday |

The team chose **Lambda container images** instead of zip packages because the ingest libraries (pandas, pyarrow, numpy, yfinance) exceed the 250 MB limit of a zip package. Unpacked, the ingest image is about 0.9 GB, far below the 10 GB Lambda limit.

The base image is `public.ecr.aws/lambda/python:3.12`, **pinned by digest** in the Dockerfile so a rebuild produces the same image. Version 3.11 is not used because its base runs Amazon Linux 2 (glibc 2.26), which cannot install numpy, pandas and pyarrow at the pinned versions.

#### Build and push the images

Images are not rebuilt when releasing to `production`: version `v1.0.1` runs ingest `5cfde32`, predict `c61abdd` and accuracy job `e31c928` (this job runs from an image in the predict repository).

Tag images with the short SHA of the commit merged into `developer`, never `latest`:

```bash
TAG=$(git rev-parse --short HEAD)
REPO=<ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com
aws ecr get-login-password --profile cpaws-dev | docker login --username AWS --password-stdin $REPO

# --provenance=false so that Lambda accepts the image
docker buildx build --platform linux/amd64 --provenance=false \
  -f docker/ingest/Dockerfile -t $REPO/cpaws-dev-ingest:$TAG --push .
docker buildx build --platform linux/amd64 --provenance=false \
  -f docker/predict/Dockerfile -t $REPO/cpaws-dev-predict:$TAG --push .
```

`.dockerignore` lets only the required code into the image: no `data/`, `tests/` or `.env`.

#### Deploy

A container Lambda can only be created once its image exists in ECR. The team deployed in steps:

1. Create the bucket and the ECR repositories first.
2. Push the ingest image, then add the ingest Lambda.
3. For predict, the stack has the flag `-c predict_repo_only=true`. The first deployment uses it to create only the repository. After the predict image is pushed, deploy again **without** the flag.

{{% notice warning %}}
Do not use `predict_repo_only=true` on later deployments: that deployment would delete the predict Lambda and the ingest destination.
{{% /notice %}}

Routine deployments:

```bash
cdk diff CpawsData --profile cpaws-dev
cdk deploy CpawsData --profile cpaws-dev
```

To try a new image before editing `app.py`, pass the tag with `-c ingest_image_tag=$TAG` or `-c predict_image_tag=$TAG`. If a later deployment does not pass it, the tag falls back to the value in `app.py`.

#### Checks after deployment

Right after the first deployment, invoke ingest in `dry_run` mode to find out whether Yahoo blocks AWS IP addresses:

```bash
aws lambda invoke --function-name cpaws-dev-ingest --profile cpaws-dev \
  --cli-binary-format raw-in-base64-out --payload '{"dry_run": true}' out.json
cat out.json   # each symbol should have about 10 rows and error = null
```

If the `dry_run` looks fine, load history with the event `{"start": "2012-01-01"}` (invoke with `--invocation-type Event`), then check that `clean/prices/dt=<date>/` exists on S3.

![ECR cpaws-dev-predict](/images/5-Workshop/5.4.1-ecr-predict.png)

*ECR repository `cpaws-dev-predict` holds 4 images, each tagged with a commit SHA, about 301 MB each.*

![Lambda functions](/images/5-Workshop/5.4.1-lambda-functions.png)

*The 6 CPAWS Lambda functions: `ingest`, `predict` and `accuracy-job` run from container images; `forecasts-api`, `accuracy-api` and `chat-api` are Python 3.12 zip packages.*

![Lambda cpaws-dev-ingest](/images/5-Workshop/5.4.1-lambda-ingest.png)

*Function `cpaws-dev-ingest` belongs to stack `CpawsData`; its Image URI points to tag `5cfde32`, architecture x86_64. The Account ID is hidden.*
