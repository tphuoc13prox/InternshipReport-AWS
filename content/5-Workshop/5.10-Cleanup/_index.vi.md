---
title: "Dọn dẹp tài nguyên"
date: 2025-09-09
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

Môi trường `dev` của nhóm vẫn chạy cho tới khi chấm xong. Các bước dưới đây dành cho người đã dựng lại workshop trong tài khoản của mình, và cũng là thứ tự nhóm sẽ làm sau khi chấm.

#### Bước 1 – Đóng đăng ký và web

1. Tự đăng ký đang mở cho thời gian chấm. Đóng lại trong vòng một ngày sau khi chấm xong, chậm nhất 10/12/2026: tắt trong `CpawsAuth` trước, sau đó build web với `VITE_ALLOW_SIGNUP=false`, rồi disable hoặc xóa các user tự tạo.
2. Trên console Amplify, xóa app `cpaws-dev-web`. App này không thuộc CDK.

#### Bước 2 – Xóa stack theo thứ tự ngược

Xóa nơi đọc trước, rồi mới xóa tài nguyên được đọc:

```bash
cdk destroy CpawsApi --profile cpaws-dev
cdk destroy CpawsOps --profile cpaws-dev
cdk destroy CpawsData --profile cpaws-dev
cdk destroy CpawsAuth --profile cpaws-dev
cdk destroy CpawsServing --profile cpaws-dev
```

Thứ gì mất, thứ gì còn:

| Tài nguyên | Removal policy | Sau `cdk destroy` |
|---|---|---|
| Lambda, schedule, API, alarm, SNS topic | mặc định | Bị xóa |
| ECR `cpaws-dev-ingest`, `cpaws-dev-predict` | `DESTROY`, tự làm rỗng khi xóa | Bị xóa cùng mọi image |
| `cpaws-dev-chat-usage` | `DESTROY` | Bị xóa |
| S3 `cpaws-dev-lake` | `RETAIN` | **Còn lại** |
| `cpaws-dev-forecasts`, `cpaws-dev-news-labels`, `cpaws-dev-accuracy` | `RETAIN` | **Còn lại** |
| User pool `cpaws-dev-users` | `RETAIN` | **Còn lại** |

#### Bước 3 – Xóa tay các tài nguyên được giữ lại

Chỉ làm khi chắc chắn không cần dữ liệu nữa.

**Bucket S3.** Bucket bật versioning, nên phải xóa hết mọi phiên bản object và delete marker trước khi xóa được bucket. Cách dễ nhất là dùng console: chọn `cpaws-dev-lake` → **Empty**, rồi **Delete**.

{{% notice warning %}}
Trong tài khoản của nhóm, một số prefix được bảo vệ bằng statement explicit deny trên `s3:DeleteObject` (artifact đánh giá). Làm rỗng bucket sẽ lỗi cho tới khi quản trị viên gỡ các statement đó khỏi bucket policy và IAM policy. Ở tài khoản mới không có các statement này thì không gặp vấn đề.
{{% /notice %}}

**DynamoDB và Cognito:**

```bash
aws dynamodb delete-table --table-name cpaws-dev-forecasts --profile cpaws-dev
aws dynamodb delete-table --table-name cpaws-dev-news-labels --profile cpaws-dev
aws dynamodb delete-table --table-name cpaws-dev-accuracy --profile cpaws-dev
aws cognito-idp delete-user-pool --user-pool-id <USER_POOL_ID> --profile cpaws-dev
```

#### Bước 4 – Tài nguyên ngoài CDK

| Tài nguyên | Cách xóa |
|---|---|
| SSM parameter `/cpaws/dev/eia-api-key` | `aws ssm delete-parameter --name /cpaws/dev/eia-api-key --profile cpaws-dev` |
| Log group CloudWatch `/aws/lambda/cpaws-dev-*` | Xóa trên console nếu còn sót (log bên trong hết hạn sau 1 tháng, nhưng bản thân log group vẫn còn) |
| AWS Budgets, Cost Anomaly Detection | Giữ tới khi có hóa đơn cuối, rồi xóa trong Billing console |
| Stack bootstrap của CDK `CDKToolkit` | Chỉ xóa nếu không còn app CDK nào khác dùng tài khoản và Region này |

#### Bước 5 – Kiểm tra

```bash
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query "StackSummaries[?starts_with(StackName, 'Cpaws')].StackName" --profile cpaws-dev
aws dynamodb list-tables --query "TableNames[?starts_with(@, 'cpaws-dev')]" --profile cpaws-dev
aws s3 ls --profile cpaws-dev | grep cpaws-dev
```

Cả ba lệnh đều không trả gì là đạt. Vài ngày sau mở Billing console để chắc chắn không còn phát sinh chi phí.
