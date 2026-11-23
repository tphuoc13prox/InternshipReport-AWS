---
title: "Chi phí"
date: 2025-09-09
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

#### Ước tính

**AWS Pricing Calculator:** [CẦN BỔ SUNG: link Pricing Calculator của nhóm]

| Dịch vụ | Tính tiền theo | Ước tính mỗi tháng |
|---|---|---:|
| [CẦN BỔ SUNG: các dòng theo dịch vụ từ bản ước tính Pricing Calculator của nhóm] | | |

#### Chi phí thực tế đến nay

Số do nhóm ghi nhận ngày 11/11/2026. Đây là số theo dõi trong quá trình làm dự án, chưa phải kỳ billing đã chốt.

| Hạng mục | Số tiền |
|---|---:|
| Chi phí thực tế 12/10 – 11/11 | khoảng 5,28 USD |
| Dự báo 12/11 – 29/11 | khoảng 1,14 USD |
| Credit còn lại ngày 11/11 | khoảng 194,67 USD |

Toàn bộ chi phí đến nay được credit AWS bao phủ. Số liệu cuối theo từng dịch vụ sẽ lấy từ Cost Explorer sau ngày 30/11.

#### Vì sao chi phí thấp

Thiết kế tránh mọi thứ tính tiền theo giờ khi không có ai dùng:

+ **Lambda chỉ chạy khi cần:** hai lượt theo lịch mỗi ngày trong tuần, predict vài giây, job accuracy khoảng 30 giây, cộng với các lần gọi API.
+ **Không endpoint chạy thường trực**, không SageMaker, không OpenSearch. Champion là mốc thống kê nên không có gì cần host.
+ **Lambda không đặt trong VPC**, nên không cần NAT Gateway, thứ mà riêng nó đã tốn hơn cả phần còn lại của hệ thống.
+ **DynamoDB on-demand**, hợp với lưu lượng thấp và không đều.
+ **Giới hạn lưu trữ:** log CloudWatch giữ 1 tháng, ECR giữ 5 image mỗi repo, bản cũ trên S3 hết hạn sau 30 ngày.

#### Khoản duy nhất có thể tăng: Bedrock

Chatbot là phần duy nhất có chi phí tăng theo số người dùng, nên có trần cứng:

| Lớp chặn | Giá trị |
|---|---|
| Chi phí mỗi câu (Haiku 4.5, đo trên benchmark) | khoảng 0,012 USD |
| Giới hạn mỗi người | 20 câu/ngày |
| Giới hạn cả hệ thống | 100 câu/ngày |
| Trường hợp xấu nhất | khoảng 1,2 USD/ngày, khoảng 36 USD/tháng |
| Throttling theo stage | 10 request/giây, burst 20 |

Trường hợp xấu nhất vẫn dưới ngân sách 50 USD/tháng của dự án. Request sai dạng không bị tính lượt, còn lỗi 5xx đã tính thì không hoàn lại, nên gọi lại liên tục cũng không đẩy chi phí lên. Hai lần chạy benchmark chính thức của prompt v4 tốn tổng cộng khoảng 0,73 USD.

#### Cảnh báo

| Cơ chế | Cấu hình |
|---|---|
| AWS Budgets | 50 USD/tháng |
| Cost Anomaly Detection | Monitor mặc định, kiểm hằng ngày, báo từ mức 100 USD tuyệt đối hoặc 40% tương đối, email đã xác nhận |

Cả hai chỉ cảnh báo; không cơ chế nào tự dừng dịch vụ khi vượt ngưỡng. Đó là lý do giới hạn lượt chatbot ở trên được thực thi bằng code.

<!-- ẢNH: Billing console, chi phí theo dịch vụ trong thời gian dự án (che Account ID) -->
[ẢNH: Billing – chi phí theo dịch vụ, đã che Account ID]
