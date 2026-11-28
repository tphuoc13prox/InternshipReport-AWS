---
title: "Dọn dẹp tài nguyên"
date: 2026-11-20
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---
#### Tổng kết

Trong workshop này, bạn đã:
- Thu thập giá theo lịch giao dịch và lưu snapshot có versioning trên S3
- Xây mốc dự báo phân vị không rò rỉ dữ liệu, đánh giá bằng pinball loss với tập test chỉ chạy một lần
- Triển khai Lambda container, DynamoDB, EventBridge Scheduler, API Gateway, Cognito bằng CDK
- Nối ingest với predict bằng Lambda Destinations và ghi dự báo đúng một lần
- Đưa chatbot Bedrock dùng tool để trả lời bằng số liệu thật, có giới hạn lượt hỏi
- Giám sát bằng CloudWatch alarm và theo dõi chi phí bằng Budgets

{{% notice info %}}
Nhóm CPAWS giữ hệ thống chạy tới hết đợt chấm cuối kỳ, sau đó mới dọn. Các bước dưới đây dành cho lúc kết thúc dự án, hoặc khi bạn dựng lại workshop trên tài khoản của mình.
{{% /notice %}}

#### Chuẩn bị và ngừng tải

1. **Xác nhận phạm vi và sao lưu.** Kiểm tra đúng tài khoản (chỉ đối chiếu 4 số cuối của Account ID), đúng region và tên stack. Lưu lại những gì cần giữ để đối chiếu: kết quả đánh giá, artifact mô hình, ảnh chụp và số liệu chi phí.
2. **Tắt hai lịch EventBridge Scheduler** (`cpaws-dev-ingest-daily`, `cpaws-dev-accuracy-daily`) trước khi xóa stack, rồi chờ các lượt Lambda đang chạy kết thúc. Như vậy không có tải mới, và không có lượt ingest nào chạy giữa lúc đang xóa.

```bash
aws sts get-caller-identity --profile cpaws-dev
aws cloudformation list-stacks --region ap-southeast-1 --profile cpaws-dev \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE --query "StackSummaries[].StackName"
```

Không dán output có Account ID hay ARN đầy đủ vào tài liệu công khai.

#### Xóa các stack

Xóa theo thứ tự ngược với lúc tạo. `CpawsApi` dùng bảng và User Pool của hai stack khác, nên phải xóa trước:

```bash
cd infra
cdk destroy CpawsApi --profile cpaws-dev
cdk destroy CpawsOps --profile cpaws-dev
cdk destroy CpawsData --profile cpaws-dev
cdk destroy CpawsServing --profile cpaws-dev
cdk destroy CpawsAuth --profile cpaws-dev
```

Xóa lần lượt từng stack, không dùng `--all` hay `--force`. Chờ stack về `DELETE_COMPLETE` (hoặc đọc lý do nếu là `DELETE_FAILED`) rồi mới sang stack tiếp theo. Ở môi trường `dev`, hai repo ECR được đặt `DESTROY` và tự làm rỗng khi xóa stack. Nghĩa là các image dùng để rollback cũng mất theo.

#### Tài nguyên `RETAIN` phải xóa tay

Một số tài nguyên được giữ lại khi xóa stack, để tránh mất dữ liệu do lỡ tay:

| Tài nguyên | Cách xóa |
|---|---|
| S3 `cpaws-dev-lake` | Bucket có versioning, nên phải xóa mọi phiên bản object rồi mới xóa được bucket (Console: **Empty**, rồi **Delete**). Trước đó, người quản trị phải gỡ các statement explicit deny chặn xóa trên `eval/` |
| DynamoDB `cpaws-dev-forecasts`, `cpaws-dev-news-labels`, `cpaws-dev-accuracy` | `aws dynamodb delete-table --table-name <TEN_BANG> --profile cpaws-dev` |
| Cognito User Pool `cpaws-dev-users` | `aws cognito-idp delete-user-pool --user-pool-id <USER_POOL_ID> --profile cpaws-dev` |
| Log group của các Lambda API | Xóa trong CloudWatch > Log Management khi không còn cần log |

Lệnh `aws s3 rb --force` không xóa các phiên bản cũ và delete marker của bucket có versioning, nên hãy dùng **Empty** trong Console. Nếu gặp explicit deny hay Object Lock thì dừng lại và báo người phụ trách dữ liệu, không tự gỡ để xóa cho nhanh.

#### Tài nguyên tạo ngoài CDK

```bash
aws ssm delete-parameter --name /cpaws/dev/eia-api-key --profile cpaws-dev
```

- Xóa app `cpaws-dev-web` trong Amplify Console.
- Nếu tài khoản còn dùng chung cho việc khác, không xóa bucket, repo và tham số của CDK bootstrap, cũng không đụng tới các KMS key do AWS quản lý.
- Giữ AWS Budgets thêm vài ngày để theo dõi xem còn phát sinh chi phí không, rồi mới xóa.

#### Kiểm tra sau khi dọn

- CloudFormation không còn stack `Cpaws*`. Stack `CDKToolkit` của bootstrap chỉ xóa nếu không còn dùng CDK trong tài khoản.
- Không còn bucket, bảng DynamoDB, User Pool hay log group nào của dự án.
- SNS topic `cpaws-dev-alarms` do stack `CpawsOps` tạo nên đã bị xóa cùng stack; chỉ cần kiểm tra lại trong SNS Console.
- Xem AWS Billing và Cost Explorer vào ngày hôm sau và sau kỳ hóa đơn kế tiếp. Số liệu billing có độ trễ, nên chi phí không về 0 ngay.
- Ghi lại danh sách tài nguyên cố ý giữ lại, lý do và người phụ trách.
