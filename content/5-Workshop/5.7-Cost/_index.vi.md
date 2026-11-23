---
title: "Chi phí và Well-Architected"
date: 2026-11-20
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---
#### Chi phí thực tế

Toàn bộ chi phí được trả bằng credit AWS của nhóm (khoảng 200 USD). Số dưới đây lấy từ AWS Billing và Cost Explorer.

| Hạng mục | Số tiền | Ghi chú |
|---|---:|---|
| Chi phí thực tế 12/10 – 11/11/2026 | khoảng 5,28 USD | Bao gồm cả giai đoạn thử nghiệm và benchmark |
| Dự báo 12/11 – 29/11/2026 | khoảng 1,14 USD | Số dự báo, chưa phải chi phí đã phát sinh |
| Một lượt benchmark chatbot chính thức | khoảng 0,73 USD | Gọi Bedrock thật, ngày 11/11/2026 |

Số chốt cuối kỳ sẽ được cập nhật sau ngày 30/11/2026.

#### Ước tính bằng AWS Pricing Calculator

Bảng ước tính: [CẬP NHẬT LINK PRICING CALCULATOR]

Ước tính dùng các giả định sau, đều là mức sử dụng thật hoặc giới hạn đã cấu hình trong hệ thống:

| Dịch vụ | Giả định sử dụng/tháng |
|---|---|
| AWS Lambda | Khoảng 22 ngày chạy × (ingest + predict + job accuracy), mỗi hàm 1.024 MB; vài nghìn lượt gọi API |
| Amazon EventBridge Scheduler | Khoảng 44 lượt gọi (2 lịch × 22 ngày) |
| Amazon S3 | Vài GB Parquet và CSV nén, có versioning |
| Amazon DynamoDB (on-demand) | 4 bản ghi dự báo mỗi ngày; đọc theo lượt xem web và lượt hỏi chatbot |
| Amazon ECR | 2 repository, tối đa 5 image mỗi repo |
| Amazon API Gateway (REST) | Vài nghìn request |
| Amazon Cognito | Dưới 50 người dùng hoạt động |
| AWS Amplify Hosting | Web tĩnh, deploy tay vài lần |
| Amazon CloudWatch | Log giữ 1 tháng, 6 alarm |
| Amazon Bedrock (Claude Haiku 4.5) | Khoảng 0,012 USD/câu; giới hạn cứng 100 câu/ngày |

**Bedrock là khoản có thể tăng nhanh nhất.** Vì vậy chatbot bắt buộc đăng nhập, có giới hạn 20 câu/người/ngày và 100 câu/ngày cho cả hệ thống. Ở mức trần đó, chi phí chatbot tối đa khoảng 1,2 USD/ngày.

#### Biện pháp tối ưu chi phí

| Biện pháp | Tác dụng |
|---|---|
| Dự báo theo lô bằng Lambda, không dùng endpoint suy luận chạy 24/7 | Mỗi ngày chỉ trả tiền cho vài giây chạy predict thay vì 730 giờ/tháng |
| Bỏ Step Functions và SageMaker AI (ADR-0011) | Không có chi phí điều phối hay huấn luyện định kỳ |
| Không đặt Lambda trong VPC, không dùng NAT Gateway | Tránh phí giờ và phí xử lý dữ liệu của NAT Gateway |
| Dự báo tính sẵn, lưu DynamoDB on-demand, đọc bằng `Query` với `Limit=1` | Mỗi lượt xem chỉ tốn một lần đọc nhỏ |
| Suy luận bằng NumPy trong Lambda predict, không dùng PyTorch | Image nhỏ, khởi động nhanh, ít GB-giây |
| Giới hạn lượt hỏi chatbot và `maxTokens` 1500 | Chặn chi phí Bedrock tăng bất thường |
| ECR giữ 5 image, log giữ 1 tháng, phiên bản cũ trên S3 giữ 30 ngày | Không để dữ liệu tích lũy vô thời hạn |
| AWS Budgets 50 USD/tháng và Cost Anomaly Detection | Phát hiện sớm khi chi phí tăng bất thường |
| Tag `Project`, `Owner`, `Env` trên mọi tài nguyên | Lọc chi phí theo dự án trong Cost Explorer |

#### Đối chiếu với AWS Well-Architected

| Trụ cột | Đã làm | Còn yếu |
|---|---|---|
| **Operational Excellence** | Hạ tầng dạng code (CDK, 5 stack), `cdk diff` trước khi deploy, CI ở mọi PR, rollback bằng một lệnh deploy với tag image cũ, ADR cho mọi quyết định kiến trúc | Deploy tay từ một máy, chưa có CD; Amplify upload zip tay |
| **Security** | IAM user + group, MFA, phiên đăng nhập ngắn hạn; IAM role quyền tối thiểu cho từng Lambda; S3 chặn public, bắt buộc SSL; API key trong SSM `SecureString`; Cognito authorizer, throttling, CORS một origin; chặn xóa kết quả đánh giá bằng explicit deny | Không bật Bedrock Guardrails; group chỉ đọc đang dùng `ReadOnlyAccess` rộng hơn nhu cầu |
| **Reliability** | Ingest lỗi một nguồn vẫn phát hành các tài sản còn lại; ghi có điều kiện nên chạy lại an toàn; bucket có versioning; bảng dự báo và User Pool `RETAIN`; 6 alarm gửi email | Một region, một môi trường; chưa có cảnh báo khi lịch không chạy |
| **Performance Efficiency** | Chọn Lambda container vì thư viện lớn; predict chạy khoảng 3,3 giây với 1.024 MB; dự báo tính sẵn nên API chỉ đọc một bản ghi | Cold start của Lambda chat cộng vào thời gian trả lời |
| **Cost Optimization** | Không có tài nguyên tính tiền theo giờ; chi phí thực tế khoảng 5,28 USD sau tháng đầu; Budgets và Cost Anomaly Detection | Chưa tách chi phí Bedrock theo tính năng |
| **Sustainability** | Chỉ chạy khi có phiên giao dịch mới (thứ Ba–thứ Bảy); không có máy chạy không tải | — |

<!-- ẢNH: Cost Explorer, chi phí theo dịch vụ (che Account ID) -->
<!-- ẢNH: AWS Pricing Calculator, trang tổng của bảng ước tính -->
