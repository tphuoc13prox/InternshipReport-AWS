---
title: "Chuẩn bị"
date: 2025-09-09
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Tài khoản AWS và Region

+ Một tài khoản AWS đã bật MFA cho root user. Chỉ dùng root user cho các việc cấp tài khoản.
+ Region **`ap-southeast-1` (Singapore)**. CDK app cố định Region này, mọi tài nguyên trong workshop đều tạo ở đây.
+ Thiết lập AWS Budgets trước khi bắt đầu. Nhóm dùng ngân sách 50 USD/tháng có email cảnh báo (xem [5.8](../5.8-cost/)).

#### Quyền dùng model trên Amazon Bedrock

Chatbot gọi **Claude Haiku 4.5** qua **global cross-Region inference profile** từ `ap-southeast-1`:

```text
global.anthropic.claude-haiku-4-5-20251001-v1:0
```

Mở Amazon Bedrock console ở `ap-southeast-1`, vào **Model access** và kiểm tra Claude Haiku 4.5 đã dùng được trong tài khoản trước khi deploy stack API.

{{% notice info %}}
Với profile global, request có thể được chuyển sang Region khác để xử lý, nên nhóm không hứa rằng câu hỏi chatbot nào cũng ở lại Singapore.
{{% /notice %}}

<!-- ẢNH: Bedrock console, trang Model access cho thấy Claude Haiku 4.5 đã dùng được (che Account ID) -->
[ẢNH: Bedrock console – Model access, Claude Haiku 4.5 đã dùng được]

#### Quyền IAM

Deploy bằng IAM identity của từng người, đăng nhập bằng credentials ngắn hạn, không dùng access key dài hạn. Trong nhóm, người deploy thuộc một group platform: được deploy hạ tầng nhưng không được quản trị IAM. Để làm workshop này, identity của bạn cần:

+ tạo và cập nhật CloudFormation stack, dùng được tài nguyên CDK bootstrap;
+ tạo IAM role và policy mà CDK sinh ra cho Lambda, Scheduler và API Gateway;
+ quản lý Lambda, ECR, S3, DynamoDB, EventBridge Scheduler, API Gateway, Cognito, CloudWatch (log, metric filter, alarm), SNS và SSM Parameter Store;
+ gọi Amazon Bedrock (`bedrock:InvokeModel`) để thử model;
+ quản lý app trên AWS Amplify Hosting.

Đăng nhập AWS CLI bằng credentials ngắn hạn, ví dụ:

```bash
aws login --profile cpaws-dev
aws sts get-caller-identity --profile cpaws-dev
```

`cpaws-dev` là tên profile nhóm dùng; bạn đặt tên nào cũng được, miễn là dùng thống nhất trong các lệnh bên dưới.

#### Công cụ

| Công cụ | Dùng để |
|---|---|
| AWS CLI v2 | Đăng nhập, login ECR, xem log và alarm |
| Python 3.12 + `venv` | Code Lambda, test và CDK app (`infra/`) |
| Node.js 22.12 trở lên + npm | AWS CDK CLI và web (`web/`) |
| AWS CDK CLI | `cdk diff`, `cdk deploy`, `cdk destroy` |
| Docker | Build image container của ingest và predict |
| Git | Clone repo của nhóm |

Lấy mã nguồn của nhóm, [CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting](https://github.com/CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting), và cài thư viện:

```bash
git clone https://github.com/CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting.git
cd CPAWS-commodity-forecasting
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements-dev.txt
pip install -r infra/requirements.txt   # aws-cdk-lib cho CDK app
python -m pytest -q                # unit test, gồm cả test synth CDK
npm install -g aws-cdk
```

Bootstrap CDK một lần cho mỗi tài khoản và Region:

```bash
cd infra
cdk bootstrap --profile cpaws-dev
```

#### Một bí mật duy nhất: API key của EIA

Lambda ingest đọc API key của EIA (chỉ dùng để đối chiếu giá dầu) từ **SSM Parameter Store** dạng `SecureString`. CDK app chỉ nhận **tên** parameter, mặc định là `/cpaws/dev/eia-api-key`. Không bao giờ để chính key trong repo, trong file `.env` hay trong giá trị context của CDK. Tạo parameter này một lần trong Systems Manager console, chọn kiểu **SecureString**.

<!-- ẢNH: Systems Manager Parameter Store, parameter /cpaws/dev/eia-api-key kiểu SecureString (ẩn giá trị) -->
[ẢNH: SSM Parameter Store – parameter `/cpaws/dev/eia-api-key`, kiểu SecureString, ẩn giá trị]
