---
title: "Daily flow"
date: 2026-11-20
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---
#### Schedule

```text
EventBridge Scheduler ── cron(15 12 ? * TUE-SAT *) UTC, retry 0
  └─► Lambda ingest (asynchronous invoke, retry 1) ── writes raw/, clean/ to S3
        └─► OnSuccess or OnFailure ─► Lambda predict (retry 0)
              ├─► DynamoDB cpaws-dev-forecasts (champion baseline-q252, conditional write)
              └─► S3 eval/shadow/s1a-l094/ (S1a shadow, not displayed)
```

12:15 UTC is **19:15 Vietnam time**, Tuesday to Saturday, i.e. after the US sessions from Monday to Friday. This time matches the publication rule: data for session D is treated as available from 12:00 UTC on day D+1. A test checks that the schedule constants in `infra/stacks/shared.py` match `src/data/config.py`.

The team **does not use Step Functions** for this flow. There are only two steps in sequence, so Lambda Destinations is enough and costs nothing extra (ADR-0011).

#### Chaining ingest to predict

Simplified from `infra/stacks/cpaws_data_stack.py`:

```python
from aws_cdk import Duration, aws_iam as iam, aws_lambda as lambda_
from aws_cdk import aws_lambda_destinations as destinations, aws_scheduler as scheduler

predict_dest = destinations.LambdaDestination(predict_fn)

ingest_fn = lambda_.DockerImageFunction(
    self, "IngestFunction",
    code=lambda_.DockerImageCode.from_ecr(ingest_repo, tag_or_digest=ingest_image_tag),
    architecture=lambda_.Architecture.X86_64,
    memory_size=1024, timeout=Duration.seconds(300),
    # retry once so Yahoo is not hit repeatedly while it is failing
    retry_attempts=1,
    # ingest still writes a snapshot for healthy assets when one source fails, so both branches call predict
    on_success=predict_dest,
    on_failure=predict_dest,
)

# Only the Scheduler of this account and this schedule group can assume the role
scheduler_role = iam.Role(self, "IngestScheduleRole",
    assumed_by=iam.PrincipalWithConditions(
        iam.ServicePrincipal("scheduler.amazonaws.com"),
        {"StringEquals": {"aws:SourceAccount": self.account, "aws:SourceArn": schedule_group_arn}},
    ),
)
scheduler_role.add_to_policy(iam.PolicyStatement(
    actions=["lambda:InvokeFunction"], resources=[ingest_fn.function_arn],
))

scheduler.CfnSchedule(self, "IngestSchedule",
    schedule_expression="cron(15 12 ? * TUE-SAT *)",
    schedule_expression_timezone="UTC",
    flexible_time_window=scheduler.CfnSchedule.FlexibleTimeWindowProperty(mode="OFF"),
    target=scheduler.CfnSchedule.TargetProperty(
        arn=ingest_fn.function_arn, role_arn=scheduler_role.role_arn, input="{}",
        retry_policy=scheduler.CfnSchedule.RetryPolicyProperty(maximum_retry_attempts=0),
    ),
)
```

Lambda Destinations only apply when the function is invoked **asynchronously**. The Scheduler always invokes asynchronously. When testing by hand, use `aws lambda invoke --invocation-type Event`.

#### Writing each forecast exactly once

Table `cpaws-dev-forecasts` has partition key `asset` and sort key `base_date`. Predict writes with a conditional `PutItem`: a published record is never overwritten, and only a sample record (`is_sample = true`) may be replaced.

```python
table.put_item(
    Item=record,
    ConditionExpression="attribute_not_exists(#a) OR #s = :t",
    ExpressionAttributeNames={"#a": "asset", "#s": "is_sample"},
    ExpressionAttributeValues={":t": True},
)
# ConditionalCheckFailedException -> log PREDICT_ALREADY_PUBLISHED, keep the existing record
```

So re-running ingest or predict on the same day cannot corrupt data, and keeping predict at 0 retries is safe.

#### Behaviour on failure

| Situation | What the system does |
|---|---|
| One price source fails | Ingest still writes the other assets and then reports an error (ingest alarm). Predict still runs for the assets with enough data |
| An asset is missing the latest session | Predict **does not fall back to an older date**: it skips that asset and reports `PredictFailed` (predict alarm) |
| S1a shadow fails | The champion is still published; the error is counted by its own metric filter |
| Re-run on the same day | `PREDICT_ALREADY_PUBLISHED`, nothing is overwritten |

#### Live measurements

The first automatic predict run took place on 27/10/2026. Since 04/11, predict runs the champion together with the S1a shadow. The first three runs with the shadow took 3.25–3.29 seconds, used about 260/1,024 MB of memory, wrote 4/4 assets, and the alarms stayed OK.

#### Checks

```bash
# latest record of one asset; only fields that contain no prices
aws dynamodb query --profile cpaws-dev --table-name cpaws-dev-forecasts \
  --key-condition-expression "asset = :a" \
  --expression-attribute-values '{":a": {"S": "gold"}}' \
  --projection-expression "asset, base_date, model_version, is_sample" \
  --no-scan-index-forward --limit 1

aws scheduler get-schedule --name cpaws-dev-ingest-daily \
  --profile cpaws-dev --query "[State,ScheduleExpression]" --output text
```

<!-- ẢNH: Console EventBridge Scheduler: lịch cpaws-dev-ingest-daily, trạng thái ENABLED -->
<!-- ẢNH: Console Lambda cpaws-dev-ingest, tab Configuration > Destinations (OnSuccess, OnFailure -> predict) -->
<!-- ẢNH: CloudWatch Logs của predict: dòng REPORT (Duration, Max Memory). KHÔNG chụp dòng PREDICT_DONE vì có giá -->
