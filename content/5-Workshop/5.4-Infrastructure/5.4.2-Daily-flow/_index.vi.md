---
title: "Luồng chạy hằng ngày"
date: 2026-11-20
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---
#### Lịch chạy

```text
EventBridge Scheduler ── cron(15 12 ? * TUE-SAT *) UTC, retry 0
  └─► Lambda ingest (gọi bất đồng bộ, retry 1) ── ghi raw/, clean/ lên S3
        └─► OnSuccess hoặc OnFailure ─► Lambda predict (retry 0)
              ├─► DynamoDB cpaws-dev-forecasts (champion baseline-q252, ghi có điều kiện)
              └─► S3 eval/shadow/s1a-l094/ (shadow S1a, không hiển thị)
```

12:15 UTC là **19:15 giờ Việt Nam**, thứ Ba đến thứ Bảy, tức sau các phiên Mỹ từ thứ Hai đến thứ Sáu. Giờ này khớp với quy tắc thời điểm công bố: dữ liệu phiên D chỉ được coi là có từ 12:00 UTC ngày D+1. Một test kiểm tra hằng số lịch trong `infra/stacks/shared.py` phải khớp với `src/data/config.py`.

Nhóm **không dùng Step Functions** cho luồng này. Chỉ có hai bước nối tiếp, nên Lambda Destinations là đủ và không tốn thêm chi phí (ADR-0011).

#### Nối ingest với predict

Rút gọn từ `infra/stacks/cpaws_data_stack.py`:

```python
from aws_cdk import Duration, aws_iam as iam, aws_lambda as lambda_
from aws_cdk import aws_lambda_destinations as destinations, aws_scheduler as scheduler

predict_dest = destinations.LambdaDestination(predict_fn)

ingest_fn = lambda_.DockerImageFunction(
    self, "IngestFunction",
    code=lambda_.DockerImageCode.from_ecr(ingest_repo, tag_or_digest=ingest_image_tag),
    architecture=lambda_.Architecture.X86_64,
    memory_size=1024, timeout=Duration.seconds(300),
    # retry 1 lần để không gọi Yahoo nhiều lần khi nguồn đang lỗi
    retry_attempts=1,
    # ingest lỗi một nguồn vẫn ghi snapshot cho tài sản tốt, nên cả hai nhánh đều gọi predict
    on_success=predict_dest,
    on_failure=predict_dest,
)

# Chỉ Scheduler của đúng tài khoản và đúng schedule group mới assume được role này
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

Lambda Destinations chỉ áp dụng khi hàm được gọi **bất đồng bộ**. Scheduler luôn gọi bất đồng bộ. Khi gọi tay để thử, dùng `aws lambda invoke --invocation-type Event`.

#### Ghi dự báo đúng một lần

Bảng `cpaws-dev-forecasts` có partition key `asset` và sort key `base_date`. Predict ghi bằng `PutItem` có điều kiện: bản đã phát hành không bị ghi đè, chỉ bản dữ liệu mẫu (`is_sample = true`) được thay.

```python
table.put_item(
    Item=record,
    ConditionExpression="attribute_not_exists(#a) OR #s = :t",
    ExpressionAttributeNames={"#a": "asset", "#s": "is_sample"},
    ExpressionAttributeValues={":t": True},
)
# ConditionalCheckFailedException -> log PREDICT_ALREADY_PUBLISHED, giữ nguyên bản cũ
```

Vì vậy chạy lại ingest hay predict trong cùng ngày không làm hỏng dữ liệu, và predict để retry 0 là an toàn.

#### Hành vi khi có lỗi

| Tình huống | Hệ thống làm gì |
|---|---|
| Một nguồn giá lỗi | Ingest vẫn ghi các tài sản khác rồi báo lỗi (alarm ingest). Predict vẫn chạy cho các tài sản đủ dữ liệu |
| Tài sản thiếu phiên mới nhất | Predict **không lùi về ngày cũ**: bỏ qua tài sản đó, báo `PredictFailed` (alarm predict) |
| Shadow S1a lỗi | Champion vẫn phát hành bình thường; lỗi được đếm bằng metric filter riêng |
| Chạy lại trong ngày | `PREDICT_ALREADY_PUBLISHED`, không ghi đè |

#### Số đo chạy thật

Lượt predict tự động đầu tiên chạy ngày 27/10/2026. Từ 04/11, predict chạy champion kèm shadow S1a. Ba lượt đầu có kèm shadow mất 3,25–3,29 giây, dùng khoảng 260/1.024 MB bộ nhớ, ghi đủ 4/4 tài sản, alarm ở trạng thái OK.

#### Kiểm tra

```bash
# bản ghi mới nhất của một tài sản; chỉ lấy các trường không chứa giá
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
