---
title: "Ingest and predict"
date: 2025-09-09
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

#### What `CpawsData` creates

| Resource | Configuration |
|---|---|
| S3 `cpaws-dev-lake` | See [5.4.1](../5.4.1-storage-and-tables/) |
| ECR `cpaws-dev-ingest`, `cpaws-dev-predict` | Scan on push, keep the 5 newest images |
| Lambda `cpaws-dev-ingest` | Container, x86_64, 1,024 MB, 300 s, async retry 1 |
| Lambda `cpaws-dev-predict` | Container, x86_64, 1,024 MB, 120 s, retry 0 |
| Lambda `cpaws-dev-accuracy-job` | Same image as predict, own command, 1,024 MB, 300 s, retry 0 ([5.4.3](../5.4.3-rolling-accuracy/)) |
| EventBridge Scheduler `cpaws-dev-ingest-daily` | `cron(15 12 ? * TUE-SAT *)` UTC, retry 0 |

None of the functions run inside a VPC, so no NAT Gateway is needed.

#### Step 1 – Build the images

The Dockerfiles are in `docker/`, and the base image `public.ecr.aws/lambda/python:3.12` is pinned by digest. Build from the repository root:

```bash
docker buildx build --platform linux/amd64 --provenance=false \
  -f docker/ingest/Dockerfile -t cpaws-ingest:dev .
docker buildx build --platform linux/amd64 --provenance=false \
  -f docker/predict/Dockerfile -t cpaws-predict:dev .
```

`--provenance=false` matters: without it buildx produces a manifest list that Lambda does not accept.

#### Step 2 – Create the predict repository, then push

A container Lambda can only be created when its image is already in ECR. For the very first deployment, create the predict repository alone:

```bash
cdk deploy CpawsData -c predict_repo_only=true --profile cpaws-dev
```

Then log in to ECR and push, tagging with the short commit SHA (`<ACCOUNT_ID>` is your own account):

```bash
aws ecr get-login-password --region ap-southeast-1 --profile cpaws-dev | \
  docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com
docker tag cpaws-predict:dev <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com/cpaws-dev-predict:<SHA>
docker push <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com/cpaws-dev-predict:<SHA>
```

Do the same for the ingest image and `cpaws-dev-ingest`. Then put the tags in `DEFAULT_INGEST_IMAGE_TAG`, `DEFAULT_PREDICT_IMAGE_TAG` and `DEFAULT_ACCURACY_IMAGE_TAG` in `infra/app.py` (the team does this through a pull request).

{{% notice warning %}}
Use `predict_repo_only=true` only once. Deploying with it later deletes the predict Lambda and the ingest destination.
{{% /notice %}}

#### Step 3 – Deploy the full stack

```bash
cdk diff CpawsData --profile cpaws-dev
cdk deploy CpawsData --profile cpaws-dev
```

In the diff, look for unexpected IAM changes. The team also checks that the diff does not contain `AWS::S3::BucketPolicy`, because one statement in the bucket policy was added by hand and CDK would remove it.

#### How the chain works

```
EventBridge Scheduler (19:15 VN, Tue–Sat)
  └─► Lambda ingest ── writes raw/ and clean/prices/dt=<date>/ to S3
        └─► OnSuccess or OnFailure ─► Lambda predict
              ├─► cpaws-dev-forecasts   (champion baseline-q252, conditional PutItem)
              └─► eval/shadow/s1a-l094/ (shadow S1a, hidden from users)
```

Three design points decide how this behaves when something goes wrong:

+ **Partial data still produces forecasts.** Ingest calls predict on failure as well. Predict publishes for every asset that has the newest session and skips the others, logging `PredictFailed`.
+ **Re-runs are safe.** A forecast already published for the same asset and base date is never overwritten (`PREDICT_ALREADY_PUBLISHED`). That is why predict has no retry, and ingest only one, so a failing price source is not called over and over.
+ **The shadow cannot block the champion.** If S1a fails, the error is logged and counted by a metric filter; the champion is still published.

No Step Functions and no SageMaker are needed: two functions joined by a destination are enough, and the champion is a statistical baseline with no model artifact.

#### Step 4 – Test without waiting for 19:15

Invoke ingest asynchronously so the destination calls predict, exactly like the schedule does:

```bash
aws lambda invoke --function-name cpaws-dev-ingest --invocation-type Event \
  --payload '{}' --cli-binary-format raw-in-base64-out --profile cpaws-dev out.json
```

Then read the predict log:

```bash
aws logs tail /aws/lambda/cpaws-dev-predict --since 90m --format short --profile cpaws-dev \
  --filter-pattern "?INIT_REPORT ?REPORT ?PREDICT_DONE ?PREDICT_SHADOW ?PREDICT_FAILED ?PREDICT_ALREADY_PUBLISHED ?PREDICT_SKIPPED"
aws s3 ls s3://cpaws-dev-lake/eval/shadow/s1a-l094/dt=YYYY-MM-DD/ --profile cpaws-dev
```

On a day that was already published, expect `PREDICT_ALREADY_PUBLISHED` instead of a new write. The `PREDICT_DONE` line contains the latest price and probability of rising, so do not paste it into a public repo or pull request.

<!-- IMAGE: CloudWatch Logs, predict log with PREDICT_DONE and REPORT lines (price values blurred) -->
[ẢNH: CloudWatch Logs – predict log with `PREDICT_DONE` and `REPORT`]

#### Measured runs

Predict first ran on schedule on 27/10/2026. Starting 04/11, image `c61abdd` computes both the champion and the S1a shadow in the same run. The acceptance rule was that the longest of the first three runs should not exceed the old 2.9 s by 2 s or more:

| Run (19:15) | Duration | Max memory | Shadow files |
|---|---:|---:|---:|
| 04/11 | 3.29 s | — | 4/4 |
| 05/11 | 3.25 s | 260 / 1,024 MB | 4/4 |
| 06/11 | 3.27 s | 258 / 1,024 MB | 4/4 |

The longest run was 0.39 s above the old baseline, so the shadow stayed on.
