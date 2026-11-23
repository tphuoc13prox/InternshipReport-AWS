---
title: "Triển khai hạ tầng với CDK"
date: 2026-11-20
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---
#### Tổng quan

Hạ tầng được viết bằng **AWS CDK (Python)** trong thư mục `infra/`, chia thành 5 stack nhỏ để deploy và review riêng từng phần. CDK synth ra template **AWS CloudFormation**, nên mọi thay đổi đều xem trước được bằng `cdk diff`.

1. [Bootstrap và stack dữ liệu](5.4.1-data-stack/): S3, ECR, Lambda container
2. [Luồng chạy hằng ngày](5.4.2-daily-flow/): Scheduler → ingest → predict → DynamoDB
3. [API, xác thực và web](5.4.3-api-cognito/): API Gateway, Cognito, Amplify Hosting

#### Thứ tự deploy

Khi một thay đổi động tới nhiều stack, deploy theo thứ tự: `CpawsServing` (bảng) → `CpawsData` (Lambda, Scheduler) → `CpawsOps` (alarm) → `CpawsApi` (route). Tài nguyên phải có trước, rồi mới tới thành phần đọc nó. `CpawsAuth` chỉ cần deploy một lần, trước `CpawsApi`.

#### Tham số `-c` không được lưu lại

`infra/app.py` đọc tham số bằng `try_get_context`. Lần deploy nào **không truyền** tham số thì giá trị quay về mặc định trong `app.py`. Hai tham số phải truyền ở **mọi** lần deploy:

| Stack | Tham số | Nếu quên |
|---|---|---|
| `CpawsOps` | `-c alarm_email=<EMAIL_NHAN_CANH_BAO>` | Subscription email bị gỡ, alarm không còn gửi email |
| `CpawsApi` | `-c allowed_origin=<ORIGIN_WEB>` | CORS quay về `http://localhost:5173`, web trên Amplify không gọi được API |

Tag image (`ingest_image_tag`, `predict_image_tag`) và phiên bản mô hình được ghi thành giá trị mặc định trong `app.py` qua PR. Nhờ vậy, lần deploy nào cũng ra cùng một cấu hình.

#### Nguyên tắc chi phí và bảo mật

- **Không có tài nguyên tính tiền theo giờ:** không endpoint suy luận, không NAT Gateway, không OpenSearch.
- **Không Lambda nào chạy trong VPC.** Ingest gọi Yahoo và EIA qua Internet công cộng, nên không cần NAT Gateway.
- **Mỗi Lambda có IAM role riêng với quyền tối thiểu.** Ví dụ: ingest chỉ được đọc và ghi `raw/*`, `clean/*` và chỉ được `ssm:GetParameter` đúng một tham số; Scheduler chỉ được `lambda:InvokeFunction` đúng hàm ingest.
- **Gắn tag `Project`, `Owner`, `Env`** cho mọi tài nguyên để lọc chi phí.
- **Thay đổi IAM và hạ tầng phải qua Pull Request** và có người phụ trách bảo mật review trước khi merge. Chỉ deploy từ nhánh `developer` sau khi PR đã merge.
- **CI không deploy.** GitHub không giữ quyền AWS; deploy chạy tay từ máy của người phụ trách hạ tầng, mỗi lần một stack.
