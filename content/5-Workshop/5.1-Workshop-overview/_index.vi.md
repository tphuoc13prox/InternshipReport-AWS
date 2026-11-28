---
title: "Giới thiệu"
date: 2026-11-20
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---
#### Bài toán

Giá vàng, bạc và dầu thô biến động mạnh khi có sự kiện lớn. Người dùng phổ thông gặp ba khó khăn:

- **Khó biết giá sắp tới dao động trong khoảng nào:** nhận định trên báo chí thường là định tính, không có con số và không công bố độ chính xác trong quá khứ.
- **Khó hiểu vì sao giá biến động:** tin tức nhiều và rời rạc.
- **Khó liên hệ với tiền Việt:** giá thế giới tính bằng USD/ounce hay USD/thùng.

#### Giải pháp

CPAWS không dự báo một con số giá duy nhất. Hệ thống dự báo **phân phối log return h phiên tới** (h = 1–7) dưới dạng 7 phân vị (0,05 đến 0,95), rồi quy đổi thành:

- **Khoảng 80%** (từ phân vị 10% đến 90%) và trung vị, hiển thị theo USD và VND;
- **Xác suất tăng** tại mỗi kỳ hạn;
- **Độ tin cậy đo trên dữ liệu chạy thật** (`rolling_8w`): tỷ lệ giá thật rơi vào khoảng 80% trong 8 tuần gần nhất.

Chatbot tiếng Việt trả lời câu hỏi về các dự báo này. Mọi con số do tool đọc từ cơ sở dữ liệu, mọi tin tức có trích nguồn, và chatbot không tư vấn đầu tư.

Mô hình đang phát hành (`baseline-q252`) là **mốc thống kê 252 phiên**: phân vị thực nghiệm của return trong khoảng một năm giao dịch gần nhất. Nhóm đã thử các mô hình học máy (DLinear, CEEMDAN, mô hình theo biến động), nhưng chưa mô hình nào vượt mốc này đủ chắc chắn theo tiêu chí chốt trước (mục [5.3.2](../5.3-data-and-model/5.3.2-baseline-forecast/)). Vì vậy web và chatbot luôn ghi rõ nguồn là "Mốc thống kê 252 phiên".

#### Kiến trúc hiện trạng

![Kiến trúc CPAWS hiện trạng](/images/5-Workshop/CPAWS-architecture-current.png)

*Sơ đồ do nhóm CPAWS vẽ bằng draw.io (Khải chỉnh lại bố cục ngày 17/11/2026). Mọi thành phần trên sơ đồ đã được triển khai và đang chạy.*

Hệ thống gồm ba luồng:

1. **Thu thập và dự báo hằng ngày** (19:15 giờ Việt Nam, thứ Ba–thứ Bảy). EventBridge Scheduler gọi Lambda `ingest` tải giá và ghi snapshot lên S3. Khi ingest kết thúc, dù thành công hay lỗi, Lambda Destinations gọi Lambda `predict`. Predict tính dự báo cho từng tài sản và ghi vào DynamoDB `cpaws-dev-forecasts`. Mỗi bản đã phát hành được ghi đúng một lần, không bị ghi đè.
2. **Đo độ tin cậy** (20:45 giờ Việt Nam). Lambda `accuracy-job` so dự báo cũ với giá thật đã biết, rồi ghi kết quả vào bảng `cpaws-dev-accuracy`.
3. **Phục vụ người dùng.** Web React trên Amplify Hosting đăng nhập bằng Cognito, gọi REST API (API Gateway + Cognito authorizer) để lấy dự báo (`/forecasts`), độ tin cậy (`/accuracy`) và hỏi chatbot (`/chat`). Lambda chat gọi Claude Haiku 4.5 trên Bedrock với hai tool `get_forecast` và `get_news`.

Các phần vận hành chạy song song với ba luồng trên:
- CloudWatch Logs, metric filter và 6 alarm gửi email qua Amazon SNS;
- API key của EIA lưu trong SSM Parameter Store;
- AWS Budgets theo dõi chi phí;
- hạ tầng viết bằng CDK, chia thành 5 stack.

#### Các stack CDK

| Stack | Tài nguyên chính |
|---|---|
| `CpawsData` | S3 `cpaws-dev-lake`, ECR `cpaws-dev-ingest` và `cpaws-dev-predict`, Lambda `ingest`, `predict`, `accuracy-job`, hai lịch EventBridge Scheduler |
| `CpawsServing` | DynamoDB `forecasts`, `news-labels`, `chat-usage`, `accuracy` (on-demand) |
| `CpawsAuth` | Cognito User Pool có xác minh email; tạm mở tự đăng ký trong thời gian chấm (đóng chậm nhất ngày 10/12/2026) |
| `CpawsApi` | REST API stage `dev`; Lambda `forecasts-api`, `accuracy-api`, `chat-api` |
| `CpawsOps` | SNS topic cảnh báo, 6 CloudWatch alarm |

Amplify Hosting, tham số SSM chứa API key và AWS Budgets được tạo ngoài CDK (xem [5.4](../5.4-infrastructure/)).

#### So với bản đề xuất

Trong quá trình làm, kiến trúc được rút gọn so với [bản đề xuất](../../2-proposal/). Mỗi thay đổi có ADR trong repo.

| Bản đề xuất | Hiện trạng | Lý do |
|---|---|---|
| Step Functions và SageMaker AI để huấn luyện lại hằng tuần | Hai Lambda nối bằng destination, không huấn luyện lại | Mốc 252 phiên không cần artifact mô hình; mô hình học máy chưa vượt mốc (ADR-0011, ADR-0012) |
| Secrets Manager | SSM Parameter Store `SecureString` | Chỉ có một key, không cần xoay vòng tự động; chi phí gần như bằng 0 |
| Gắn nhãn tin bằng Bedrock (Nova 2 Lite) | Chỉ gắn nhãn tay thông cáo Fed/EIA trong thời gian phát triển; sản phẩm đã đóng băng nên bộ tin không cập nhật thêm | Lần kiểm trên 49 thông cáo thật không đạt ngưỡng (ADR-0013) |
| Router chọn giữa Nova và Haiku | Mọi câu hỏi đi Claude Haiku 4.5 | Router không đạt ngưỡng trên bộ câu giữ riêng (mục [5.5](../5.5-chatbot/)) |
| Bedrock Guardrails, Knowledge Bases | Không bật | Kiểm soát bằng quy tắc prompt, tool và kiểm tra phía server (quyết định ngày 11/11/2026) |

#### Demo sản phẩm

- Video demo: [https://www.youtube.com/watch?v=useqd8Wokqw](https://www.youtube.com/watch?v=useqd8Wokqw)
- Production (Amplify Hosting): [https://dev.d2qw1ouvpqt0l9.amplifyapp.com/](https://dev.d2qw1ouvpqt0l9.amplifyapp.com/). Trang giới thiệu xem được khi chưa đăng nhập; trang dự báo, độ chính xác, hướng dẫn đọc và chatbot cần tài khoản. Trong thời gian chấm, người chấm tự tạo tài khoản bằng nút **Tạo tài khoản** (xác minh qua email).
- Mã nguồn: [CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting](https://github.com/CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting), nhánh `production` (tag `v1.0.1`).
