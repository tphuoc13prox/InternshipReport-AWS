---
title: "Worklog Tuần 3"
date: 2026-09-28
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:
- Thiết lập Hybrid DNS giữa AWS và on-premises với Route 53 Resolver.
- Kết nối nhiều VPC bằng VPC Peering, Transit Gateway và tìm hiểu sâu hơn về networking trên AWS.
- Tham gia buổi watch party AWS Cloud and AI Day Hanoi tại TP.HCM và buổi fireside chat với Dr. Werner Vogels.
- Họp team project để hoàn thiện ý tưởng, chốt đề tài và phân vai trò trong nhóm.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 | **- Thực hành lab: Hybrid DNS với Route 53 Resolver**<br>+ Chuẩn bị: tạo key pair, chạy CloudFormation template, cấu hình security group<br>+ Kết nối tới Remote Desktop Gateway (RDGW)<br>+ Triển khai AWS Managed Microsoft AD<br>+ Thiết lập DNS:<br>* Tạo Route 53 Outbound Endpoint và Resolver Rules<br>* Tạo Route 53 Inbound Endpoints<br>* Kiểm tra phân giải tên miền hai chiều<br>+ Dọn dẹp tài nguyên | 28/09/2026 | 28/09/2026 | [Thiết lập Hybrid DNS với Route 53 Resolver](https://000010.awsfcaj.com/vi/) |
| 3 | **- Tham gia Sự kiện 2: AWS Cloud and AI Day Hanoi - Watch Party: Ho Chi Minh City**<br>**- Họp team project: Hoàn thiện idea, bổ sung thêm các tính năng dự kiến**<br>+ Chốt đề tài CPAWS: nền tảng web dự đoán giá vàng và bạc (quy đổi từ quỹ ETF GLD/SLV), dầu Brent và WTI 1–7 ngày tới, hiển thị theo USD và VND<br>+ Bổ sung chatbot tiếng Việt trên Amazon Bedrock giải thích dự đoán, có trích nguồn<br>+ Phân vai trò trong team: tôi phụ trách News & Chatbot (gắn nhãn tin tức, chatbot) | 29/09/2026 | 29/09/2026 | [Sự kiện 2](../../4-eventparticipated/4.2-event2/)<br>[Amazon Bedrock](https://aws.amazon.com/bedrock/) |
| 4 | **- Thực hành lab: VPC Peering và AWS Transit Gateway**<br>+ VPC Peering: chạy CloudFormation template, tạo security group và EC2, cập nhật network ACL, tạo peering connection, bật cross-peer DNS, dọn dẹp tài nguyên<br>+ Transit Gateway: tạo Transit Gateway, attachment và route table, thêm route Transit Gateway vào route table của VPC, dọn dẹp tài nguyên | 30/09/2026 | 30/09/2026 | [Thiết lập VPC Peering](https://000019.awsfcaj.com/vi/)<br>[Thiết lập AWS Transit Gateway](https://000020.awsfcaj.com/vi/) |
| 5 | **- Thực hành lab: AWS Networking and Content Delivery (phần 1)**<br>+ Chuẩn bị: triển khai các VPC, chấp nhận điều khoản Cisco CSR<br>+ Tìm hiểu sâu các thành phần của VPC<br>+ Transit Gateway và Site-to-Site VPN:<br>* Truy cập router Cisco CSR<br>* Thiết lập Site-to-Site VPN và thêm đường ECMP<br>* Định tuyến qua Transit Gateway<br>+ Route 53 DNS endpoint và private hosted zone: triển khai và kiểm tra DNS | 01/10/2026 | 01/10/2026 | [AWS Networking and Content Delivery](https://000092.awsfcaj.com/vi/) |
| 6 | **- Tham gia Sự kiện 3: Fireside Chat với Dr. Werner Vogels (buổi sáng)**<br>+ Bài học vận hành của Amazon, câu chuyện ra đời Dynamo và DynamoDB, hình mẫu "Renaissance Developer"<br>**- Thực hành lab: AWS Networking and Content Delivery (phần 2)**<br>+ VPC endpoint cho các dịch vụ AWS: triển khai và kiểm tra VPC endpoint<br>+ VPC endpoint service (AWS PrivateLink)<br>+ VPC Peering: gửi yêu cầu peering, cấu hình định tuyến, kiểm tra kết nối<br>+ Transit Gateway Network Manager: thiết lập Network Manager, cấu hình site, Network Insights<br>+ Dọn dẹp tài nguyên | 02/10/2026 | 02/10/2026 | [AWS Networking and Content Delivery](https://000092.awsfcaj.com/vi/)<br>[Sự kiện 3](../../4-eventparticipated/4.3-event3/) |

### Kết quả đạt được tuần 3:
1. Thiết lập Hybrid DNS với Route 53 Resolver inbound/outbound endpoint và phân giải tên miền được theo cả hai chiều.
2. Kết nối các VPC bằng VPC Peering và Transit Gateway, thực hành VPC endpoint, PrivateLink và Network Manager.
3. Tham gia [buổi watch party AWS Cloud and AI Day Hanoi tại TP.HCM](../../4-eventparticipated/4.2-event2/), thứ 3 (29/09) dành cho sự kiện và buổi họp team project nên hôm đó không có nội dung học; sáng 02/10 tham gia thêm buổi [fireside chat với Dr. Werner Vogels](../../4-eventparticipated/4.3-event3/).
4. Cùng nhóm chốt đề tài CPAWS (dự báo giá hàng hóa có chatbot) và nhận phần News & Chatbot.
