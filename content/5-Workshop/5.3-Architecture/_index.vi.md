---
title: "Kiến trúc và luồng dữ liệu"
date: 2025-09-09
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Kiến trúc hiện trạng

![Kiến trúc hiện trạng CPAWS](/images/5-Workshop/CPAWS-architecture-current.png)

*Hình 5.1 – Kiến trúc CPAWS đã triển khai, cập nhật ngày 13/11/2026. Người vẽ: Châu Trần Mạnh Tiến (nhóm CPAWS), vẽ bằng draw.io với bộ AWS Architecture Icons chính thức. Mọi thành phần trên sơ đồ đều đã deploy và đang chạy.*

Toàn bộ hệ thống nằm trong một tài khoản AWS, Region `ap-southeast-1`, môi trường `dev`. Hệ thống hoàn toàn serverless: không có máy chủ chạy cả ngày, không Lambda nào đặt trong VPC, không có NAT Gateway. Các số thứ tự dưới đây khớp với số trên mũi tên của sơ đồ.

#### Luồng 1 – Thu thập và dự báo hằng ngày (thứ Ba đến thứ Bảy, 19:15 giờ Việt Nam)

1. **EventBridge Scheduler** kích hoạt theo `cron(15 12 ? * TUE-SAT *)` (giờ UTC) và gọi Lambda ingest bất đồng bộ.
2. **Lambda ingest** (image container) tải giá GLD, SLV, BZ=F, CL=F và tỷ giá USD/VND; dữ liệu EIA và NBP chỉ dùng để đối chiếu.
3. Lambda ghi file gốc và snapshot đã làm sạch `clean/prices/dt=<ngày>/` vào **data lake S3**.
4. Ingest thành công hay thất bại, **Lambda destination** (OnSuccess và OnFailure) đều gọi Lambda predict. Một nguồn lỗi thì các tài sản đã có dữ liệu mới vẫn được phát hành.
5. **Lambda predict** đọc snapshot, đồng thời tính challenger shadow S1a và ghi kết quả này vào `eval/shadow/` trên S3 (người dùng không thấy).
6. Dự báo phát hành được ghi vào bảng DynamoDB **`cpaws-dev-forecasts`** bằng `PutItem` có điều kiện, nên dự báo đã phát hành không bao giờ bị ghi đè.

Khung 19:15 được chọn có lý do. Backtest giả định dữ liệu của một phiên chỉ dùng được từ 12:00 UTC hôm sau, và lịch chạy theo đúng giả định đó, nên hệ thống thật không bao giờ dùng thông tin mà backtest không có.

#### Luồng 2 – Đo độ tin cậy (cùng các ngày, 20:45 giờ Việt Nam)

7. Lịch thứ hai, `cron(45 13 ? * TUE-SAT *)`, khởi động Lambda **job accuracy** (dùng image của predict với lệnh riêng).
8. Job lấy từng dự báo đã phát hành, đặt cạnh giá thực tế có sau đó, trong cửa sổ 56 ngày (`rolling_8w`), cho cả champion và S1a.
9. Kết quả được ghi vào **`cpaws-dev-accuracy`** bằng conditional write.

#### Luồng 3 – Dịch vụ cho người dùng

10. Người dùng mở web trên **AWS Amplify Hosting** (React + Vite).
11. Đăng nhập bằng **Amazon Cognito** và nhận ID token.
12. Trình duyệt gọi **API Gateway** (REST, stage `dev`) qua HTTPS kèm ID token. Cognito authorizer bảo vệ mọi route trừ `/health`.
13. **Lambda forecasts-api** đọc `cpaws-dev-forecasts` cho `GET /forecasts/{asset}`. Từ bản v1.0.0 (22/11), Lambda này đọc thêm lịch sử giá ngắn trên S3 (`serving/price-history`) cho biểu đồ 2 năm trên web; luồng này có sau khi vẽ sơ đồ.
14. **Lambda chat-api** (`POST /chat`) trước hết tính lượt hỏi vào bảng **`cpaws-dev-chat-usage`**.
15. Sau đó gọi **Amazon Bedrock** (Claude Haiku 4.5, Converse API) với hai tool: `get_forecast` đọc cùng bảng dự báo với trang web, `get_news` đọc tin Fed/EIA gắn nhãn tay được đóng gói cùng Lambda.
16. **Lambda accuracy-api** trả `GET /accuracy/{asset}` từ `cpaws-dev-accuracy`.

#### Vận hành và bảo mật

17. Alarm CloudWatch gửi email qua **Amazon SNS**.
18. Lập trình viên đẩy code lên GitHub (chỉ CI: lint, test, build web) và deploy stack CDK, image container từ laptop.

Khối vận hành trên sơ đồ còn có IAM role theo least privilege, SSM Parameter Store cho key EIA, Amazon ECR cho hai image, AWS Budgets, và AWS CDK → CloudFormation với 5 stack.

#### Dịch vụ AWS và lý do chọn

| Dịch vụ | Vai trò trong CPAWS | Vì sao chọn |
|---|---|---|
| Amazon EventBridge Scheduler | Chạy ingest lúc 19:15 và job accuracy lúc 20:45 | Giá chỉ đổi mỗi ngày một lần; lịch chạy là đủ và không tốn gì giữa các lượt |
| AWS Lambda (image container) | Ingest, predict, job accuracy | Thư viện Python nặng vừa trong image (tối đa 10 GB); trả tiền theo lượt chạy; mỗi lượt chỉ vài giây |
| AWS Lambda (zip, Python 3.12) | forecasts-api, accuracy-api, chat-api | Xử lý request/response ngắn, tự mở rộng |
| Amazon ECR | Lưu image ingest và predict | Image gắn tag bằng SHA ngắn của commit, rollback chỉ cần đổi một tham số |
| Amazon S3 | Data lake: dữ liệu gốc, snapshot đã làm sạch, file shadow và đánh giá | Lưu trữ rẻ, bền; versioning chống ghi đè |
| Amazon DynamoDB (on-demand) | Dự báo, nhãn tin, đếm lượt chat, độ tin cậy | Đọc theo khóa nhanh; tính tiền on-demand hợp với lưu lượng thấp, không đều |
| Amazon API Gateway (REST) | Một API cho dự báo, độ tin cậy và chat | Có sẵn Cognito authorizer và throttling theo stage |
| Amazon Cognito | User pool cho web | Đăng nhập được quản lý sẵn; API chỉ nhận ID token hợp lệ |
| Amazon Bedrock | Claude Haiku 4.5 cho chatbot | Không phải thuê GPU; tool use giữ con số gắn với database |
| AWS Amplify Hosting | Chạy web single-page | Hosting tĩnh có rewrite cho route của SPA |
| Amazon CloudWatch + Amazon SNS | Log (giữ 1 tháng), metric filter, 6 alarm, email | Có lỗi là nhóm biết mà không cần ai ngồi canh console |
| AWS Systems Manager Parameter Store | Key EIA dạng `SecureString` | Chỉ có một bí mật nên không cần Secrets Manager |
| AWS Budgets + Cost Anomaly Detection | Cảnh báo chi phí | Báo sớm khi chi phí bất thường |
| AWS CDK / CloudFormation | 5 stack | Cả hệ thống review được trong pull request, dựng lại hoặc xóa bằng lệnh |

Những gì nhóm quyết định **không** dùng: Step Functions và SageMaker (không có mô hình học nào cần huấn luyện lại, xem [5.11](../5.11-lessons-learned/)), endpoint chạy thường trực, VPC có NAT Gateway, OpenSearch, Bedrock Knowledge Bases, và Bedrock Guardrails (không bật, xem [5.5.2](../5.5-api-and-chatbot/5.5.2-bedrock-chatbot/)).

#### Đối chiếu Well-Architected

**Operational Excellence (Vận hành xuất sắc)**
+ Toàn bộ hạ tầng là code (5 stack CDK); mọi thay đổi đi qua pull request có ít nhất một approve và CI (`ruff`, `pytest` kèm test synth CDK, typecheck/test/build của web).
+ 6 alarm gửi email cho nhóm, và nhóm kiểm tay sau mỗi lượt 19:15.
+ Mỗi thay đổi có rủi ro đều có cách rollback bằng một lệnh (mục [5.7](../5.7-monitoring-and-testing/)).
+ Các quyết định thiết kế được ghi thành ADR trong repo.

**Security (Bảo mật)**
+ API chỉ nhận ID token của Cognito; throttling 10 request/giây, burst 20; CORS chỉ cho một origin web.
+ Mỗi role của Lambda chỉ có đúng các quyền nó cần. Ví dụ Lambda chat được `Query` bảng dự báo, `UpdateItem`/`GetItem` trên bảng đếm lượt và `bedrock:InvokeModel` trên đúng ARN của model và profile, không có quyền streaming.
+ Bí mật duy nhất nằm trong SSM Parameter Store; thành viên dùng MFA và credentials CLI ngắn hạn; pre-commit hook quét bí mật vì repo để public.
+ Bucket S3 chặn truy cập công khai và bắt buộc SSL. Câu trả lời và thông báo lỗi của chatbot không chứa Account ID, ARN, tên bảng, model ID, system prompt hay stack trace.

**Reliability (Độ tin cậy)**
+ Predict và job accuracy ghi có điều kiện, nên chạy lại không làm hỏng dữ liệu đã phát hành. Predict để retry 0, ingest retry 1, để không gọi dồn vào nguồn giá khi nguồn đang lỗi.
+ Ingest lỗi vẫn gọi predict; mô hình shadow hỏng không chặn được champion.
+ Bucket S3 có versioning (giữ bản cũ 30 ngày), bảng dự báo và user pool để `RETAIN`.
+ Chatbot tự tính hạn chót từ mốc 29 giây của API Gateway và trả `504` gọn gàng thay vì bị cắt ngang.

**Performance Efficiency (Hiệu năng)**
+ Dự báo tính theo lô mỗi ngày một lần và đọc theo khóa, mở trang không bao giờ phải chạy mô hình.
+ Số đo thực tế: predict khoảng 3,3 giây, dùng khoảng 260 MB trên 1.024 MB; job accuracy 30,3 giây; chatbot trả lời trung vị 4,0 giây, p90 5,7 giây trên benchmark.

**Cost Optimization (Tối ưu chi phí)**
+ Không có gì tính tiền theo giờ: không endpoint, không NAT Gateway, không OpenSearch. DynamoDB on-demand.
+ Log giữ 1 tháng; ECR giữ 5 image mỗi repo.
+ Giới hạn lượt chatbot (20 câu/người/ngày, 100 câu/ngày cho cả hệ thống) chặn trần chi phí Bedrock. Chi tiết ở [5.8](../5.8-cost/).

**Sustainability (Bền vững)** (ghi nhận, không chấm điểm)
+ Phần tính toán chỉ chạy vài phút mỗi ngày, không có tài nguyên nào ngồi chờ lưu lượng.
