---
title: "Chi phí và Well-Architected"
date: 2026-11-20
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---
#### Chi phí thực tế

Toàn bộ chi phí được trả bằng credit AWS của nhóm (khoảng 200 USD). Số liệu dưới đây lấy từ Cost Explorer cho khoảng 01/10–24/11/2026, lọc Charge type = Usage và nhóm theo Service.

| Hạng mục | Số tiền | Ghi chú |
|---|---:|---|
| **Tổng** | **khoảng 6,35 USD** | Tháng 10: 3,00 USD; tháng 11 (tới 24/11): 3,35 USD. Đã gồm thử nghiệm và benchmark |
| Claude Haiku 4.5 (Bedrock) | 5,56 USD | Chatbot, benchmark và các lần hỏi thử |
| Bedrock (các model khác) | 0,50 USD | Chỉ có trong tháng 10, lúc thử Nova 2 Lite để gắn nhãn tin và chạy benchmark |
| Amazon ECR | 0,11 USD | Lưu image container |
| Cost Explorer | 0,07 USD | Phí truy vấn Cost Explorer |
| Amazon S3 | 0,01 USD | Data lake |
| Lambda, DynamoDB, API Gateway, Amplify, CloudWatch, Cognito, KMS, SNS, CloudFormation | dưới 0,01 USD mỗi dịch vụ | Cost Explorer làm tròn về 0,00 |

Cost Explorer còn ghi Relational Database Service (0,10 USD) và Secrets Manager. Hai khoản này đến từ một bài lab cá nhân chạy trên cùng tài khoản, không thuộc kiến trúc CPAWS: CPAWS không dùng RDS và lưu API key trong SSM Parameter Store.

![Cost Explorer](/images/5-Workshop/5.7-cost-explorer.png)

*Cost Explorer, 01/10–24/11/2026, nhóm theo Service: tổng 6,35 USD, trung bình 3,18 USD/tháng, phần lớn là Claude Haiku 4.5.*

![Cost Explorer theo dịch vụ](/images/5-Workshop/5.7-cost-breakdown.png)

*Bảng chi phí theo dịch vụ trong cùng khoảng thời gian. Dòng Relational Database Service đến từ bài lab cá nhân, nằm ngoài CPAWS.*

#### Dự toán bằng AWS Pricing Calculator

Nguyễn Tiến Long lập dự toán ngày 24/11/2026 cho **một môi trường** đang chạy (`cpaws-dev-*`, region Singapore), xem như mức vận hành production với lưu lượng thấp. Nhánh `production` trong repo không tạo thêm môi trường AWS thứ hai, nên chi phí không nhân đôi.

**Kết quả: khoảng 5,26 USD/tháng, tức 63,12 USD/năm** (số chưa làm tròn là 5,2539 USD/tháng). Dự toán gồm ba phần:

- 34 dòng lập trong Pricing Calculator của Billing Console: 1,49 USD/tháng. Từng dòng được ghi ở [Phụ lục 5.7.1](5.7.1-estimate-details/).
- Amazon Bedrock, dự toán public: [calculator.aws – Bedrock](https://calculator.aws/#/estimate?id=f81e944af4c143c007493dbfad5bfa39c934efbe) (3,77 USD/tháng).
- EventBridge Scheduler, dự toán public: [calculator.aws – EventBridge Scheduler](https://calculator.aws/#/estimate?id=1530e02ff22ccf47d5f43f5a203d3d3bc392f421) (0,00 USD/tháng, nằm trong Free Tier 14 triệu lượt gọi).

| Dịch vụ | Vai trò | USD/tháng |
|---|---|---:|
| Amazon Bedrock (Claude Haiku 4.5) | Chatbot | 3,766 |
| Amazon CloudWatch | Log, metric, 6 alarm | 1,201 |
| Amazon ECR + Data Transfer | Image container, dữ liệu truyền ra | 0,118 |
| Amazon Cognito | Đăng nhập (5 MAU) | 0,075 |
| AWS Lambda | 6 hàm | 0,048 |
| AWS Amplify | Web tĩnh | 0,013 |
| Amazon S3 | Data lake | 0,013 |
| Amazon API Gateway | REST API | 0,011 |
| Amazon DynamoDB | Bảng dự báo, độ tin cậy, bộ đếm chat | 0,009 |
| AWS KMS | Lệnh gọi API mã hóa | 0,0004 |
| Amazon SNS, EventBridge Scheduler, SSM Parameter Store | Cảnh báo, lịch, tham số | 0 |
| **Tổng** | | **5,254** |

**Giả định tải chính mỗi tháng** (đầy đủ ở phụ lục):

| Thông số | Giá trị | Cơ sở |
|---|---:|---|
| Người dùng Cognito | 5 MAU | 3 MAU trong tháng 10, 4 MAU từ 01–23/11 |
| Request API | 2.700 | Giả định mỗi ngày 60 lượt xem dự báo, 20 lượt xem độ tin cậy và 10 câu chat |
| Lượt chạy job | 66 | 22 ngày × (ingest, predict, accuracy-job) |
| Lambda | 2.766 lượt, 2.817 GB-giây | Thời gian chạy đo trong 17–23/11/2026 |
| Bedrock | 300 câu chat, khoảng 3,08 triệu token vào và 0,14 triệu token ra | Từ 7 câu chat thật ngày 22–23/11: trung bình 1,86 lần gọi model và 10.272 token vào mỗi câu; giá 1 USD/1 triệu token vào, 5 USD/1 triệu token ra |

**Khoản lớn nhất** là Bedrock (71,7%), sau đó là CloudWatch (22,9%). Khoản CloudWatch gồm 0,60 USD cho 6 alarm và 0,60 USD dự phòng cho 2 custom metric. Dự toán dùng giá trước ưu đãi cho phần lớn dịch vụ, nên các khoản nhỏ như alarm cao hơn hóa đơn thực tế.

**Độ nhạy theo số câu chat** (chỉ nhân phần token Bedrock, các khoản khác giữ nguyên):

| Kịch bản | Câu chat/tháng | Bedrock (USD/tháng) | Tổng (USD/tháng) |
|---|---:|---:|---:|
| Dự toán gốc | 300 | 3,77 | 5,25 |
| Gấp 5 lần | 1.500 | 18,83 | 20,32 |
| Dùng hết quota 100 câu/ngày | 3.000 | 37,66 | 39,14 |

Giới hạn lượt hỏi không phải giới hạn tiền: câu hỏi dài hơn hoặc gọi tool nhiều hơn vẫn làm chi phí mỗi câu tăng.

**Cognito khi mở tự đăng ký:** trong thời gian chấm, nếu 5 thành viên và tối đa 40 người được mời cùng hoạt động trong tháng (khoảng 45 MAU), phí Cognito khoảng 0,68 USD/tháng thay vì 0,075 USD.

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

Các đề xuất dưới đây **chưa làm**. Mỗi đề xuất cần đo trước và sau khi đổi; chưa đo thì không ghi là đã tiết kiệm:

- Giảm token đầu vào của chatbot (bớt ngữ cảnh lặp lại, tool chỉ trả các trường cần dùng). Giảm 20% token đầu vào thì tiết kiệm khoảng 0,62 USD/tháng, tức 12% tổng dự toán.
- Thử lại router đưa câu đơn giản sang Nova 2 Lite, và chỉ dùng nếu đạt ngưỡng trên một bộ câu mới.
- Đo lại bộ nhớ phù hợp cho từng Lambda (256/512/1.024 MB) và cân nhắc kiến trúc ARM nếu image và thư viện tương thích.
- Một log group cũ vẫn giữ log 2 năm; cần đưa về mức 1 tháng như các log group khác.
- Thêm các ngưỡng cảnh báo 50/80/100% cho budget, hợp với mức chi phí thấp hiện tại.

#### Đối chiếu với AWS Well-Architected

| Trụ cột | Đã làm | Còn yếu |
|---|---|---|
| **Operational Excellence** | Hạ tầng dạng code (CDK, 5 stack), `cdk diff` trước khi deploy, CI ở mọi PR, rollback bằng một lệnh deploy với tag image cũ, ADR cho mọi quyết định kiến trúc | Deploy tay từ một máy, chưa có CD; Amplify upload zip tay |
| **Security** | IAM user + group, MFA, phiên đăng nhập ngắn hạn; IAM role quyền tối thiểu cho từng Lambda; S3 chặn public, bắt buộc SSL; API key trong SSM `SecureString`; Cognito authorizer, throttling, CORS một origin; chặn xóa kết quả đánh giá bằng explicit deny | Không bật Bedrock Guardrails; group chỉ đọc đang dùng `ReadOnlyAccess` rộng hơn nhu cầu |
| **Reliability** | Ingest lỗi một nguồn vẫn phát hành các tài sản còn lại; ghi có điều kiện nên chạy lại an toàn; bucket có versioning; bảng dự báo và User Pool `RETAIN`; 6 alarm gửi email | Một region, một môi trường; chưa có cảnh báo khi lịch không chạy |
| **Performance Efficiency** | Chọn Lambda container vì thư viện lớn; predict chạy khoảng 3,3 giây với 1.024 MB; dự báo tính sẵn nên API chỉ đọc một bản ghi | Cold start của Lambda chat cộng vào thời gian trả lời |
| **Cost Optimization** | Không có tài nguyên tính tiền theo giờ; chi phí thực tế khoảng 6,35 USD từ 01/10 tới 24/11/2026, dự toán 5,26 USD/tháng; Budgets và Cost Anomaly Detection | Chưa tách chi phí Bedrock theo tính năng |
| **Sustainability** | Chỉ chạy khi có phiên giao dịch mới (thứ Ba–thứ Bảy); không có máy chạy không tải | — |

Các bước tắt lịch, xóa stack, xóa tài nguyên `RETAIN` và kiểm tra hóa đơn nằm ở mục [5.8](../5.8-cleanup/).
