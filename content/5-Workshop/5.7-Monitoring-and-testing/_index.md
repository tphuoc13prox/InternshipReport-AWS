---
title: "Monitoring, testing and rollback"
date: 2025-09-09
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Step 1 – Alarms (`CpawsOps`)

`CpawsOps` creates the SNS topic `cpaws-dev-alarms` and six CloudWatch alarms. They share one rule: **at least 1 in 5 minutes**, and a period with no data counts as OK.

| Alarm | Metric | Fires when |
|---|---|---|
| `cpaws-dev-ingest-errors` | `AWS/Lambda Errors` of ingest | A main price source fails or the Lambda crashes |
| `cpaws-dev-predict-errors` | `AWS/Lambda Errors` of predict | Predict had to skip at least one asset, or crashed |
| `cpaws-dev-predict-shadow-errors` | Metric filter on the predict log | S1a failed or was misconfigured. Because the champion is published anyway, the plain `Errors` metric stays at zero in this case |
| `cpaws-dev-accuracy-job-errors` | `AWS/Lambda Errors` of the accuracy job | An asset `FAILED`, timeout or crash. "Not enough data yet" is not an error |
| `cpaws-dev-chat-api-errors` | `AWS/Lambda Errors` of chat | Timeout or crash |
| `cpaws-dev-chat-api-internal-errors` | Metric filter `CPAWS/ChatInternalErrors` on the chat log | The handler caught an unexpected error and returned `500`. 4xx, 429 and Bedrock errors with their own code are designed behaviour and do not fire |

The two metric filters exist because those failures are caught inside the code, so Lambda never reports them as `Errors`.

The alarm email is passed at deploy time and never written into the repository:

```bash
cdk diff CpawsOps -c alarm_email=EMAIL_NHAN_CANH_BAO --profile cpaws-dev
cdk deploy CpawsOps -c alarm_email=EMAIL_NHAN_CANH_BAO --profile cpaws-dev
```

After the first deploy, the recipient has to click **Confirm subscription** in the email from SNS. Pass `alarm_email` on every later deploy too; without it, the subscription is removed.

```bash
aws cloudwatch describe-alarms --profile cpaws-dev \
  --query "MetricAlarms[].[AlarmName,StateValue]" --output text
```

<!-- IMAGE: CloudWatch console, Alarms list with the six cpaws-dev-* alarms in OK state -->
[ẢNH: CloudWatch – six `cpaws-dev-*` alarms in OK]

#### Step 2 – Manual check after each run

Alarms only fire on errors. If the schedule never fires, there is no data, and no data counts as OK, so nobody gets an email. For that reason the team checks by hand after every 19:15 run:

+ the predict log: `PREDICT_DONE` with `written` or `already_published`, and `Duration`;
+ the number of shadow files in `eval/shadow/s1a-l094/dt=<date>/` (expected 4);
+ the state of all alarms;
+ after 20:45, the accuracy job log and the schedule `State`.

The commands are in [5.4.2](../5.4-data-pipeline/5.4.2-ingest-and-predict/) and [5.4.3](../5.4-data-pipeline/5.4.3-rolling-accuracy/). The planned fix is an alarm on the ingest `Invocations` with a one-day period that treats missing data as breaching and fires only after three days without a run, because Sunday and Monday (UTC) have no schedule anyway.

#### Step 3 – Testing before a deploy

| Layer | What runs | Where |
|---|---|---|
| Python | `ruff check`, `ruff format --check`, `pytest -q` (job `test-and-lint`) | GitHub Actions on every push and pull request |
| Infrastructure | `tests/test_infra_*.py` synthesize the CDK app and check IAM permissions, environment variables, schedules and alarms | Same job. It installs `aws-cdk-lib` and first runs an `import aws_cdk` check, so the CDK tests can never be silently skipped and still show green |
| Web | `npm ci`, `typecheck`, `test`, `build` | Same workflow |
| Pre-commit | gitleaks and `detect-aws-credentials` (the repository is public), plus ruff, ruff-format, `check-added-large-files` and `end-of-file-fixer` | On every commit |
| Chatbot | Benchmark against real Bedrock with locked questions and thresholds | Run by hand, see [5.5.2](../5.5-api-and-chatbot/5.5.2-bedrock-chatbot/) |

CI never deploys and GitHub holds no AWS credentials. The `developer` branch needs at least one approval to merge; IAM and infrastructure changes are reviewed by the team's security lead in the pull request.

Two rules are checked when the CDK app is synthesized, so a mistake fails before anything reaches AWS:

+ a unit test compares the schedule constants in `infra/stacks/shared.py` with the data availability rule in `src/data/config.py`, which the backtest also relies on;
+ CDK refuses to synthesize if `MODEL_VERSION` differs from any value of `MODEL_VERSIONS`, so an older predict image still runs the right champion. This is also why changing the champion goes through a pull request on `app.py`.

#### Step 4 – Rollback

Every risky change has a way back with one deploy or one upload, without rebuilding an image:

| Situation | Command | Rebuild? |
|---|---|---|
| Shadow S1a fails or slows predict | `cdk deploy CpawsData -c shadow_models=off --profile cpaws-dev` | No |
| New predict image is broken | `cdk deploy CpawsData -c predict_image_tag=ce77bcb -c shadow_models=off --profile cpaws-dev` | No, old image still in ECR |
| New ingest image is broken | `cdk deploy CpawsData -c ingest_image_tag=TAG_CU --profile cpaws-dev` | No |
| Accuracy job misbehaves | `cdk deploy CpawsData -c accuracy_schedule=off --profile cpaws-dev` | No |
| Web on Amplify cannot call the API | `cdk deploy CpawsApi -c allowed_origin=http://localhost:5173 --profile cpaws-dev` | No |
| New web build is broken, or metal prices must be hidden | Upload the previous zip, or build with `VITE_SHOW_METAL_PRICES=false` | Web only |

{{% notice warning %}}
ECR keeps only the 5 newest images per repository. Before pushing a new image, check that the rollback tag is still there, or choose a new one.
{{% /notice %}}
