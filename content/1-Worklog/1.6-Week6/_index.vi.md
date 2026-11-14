---
title: "Worklog Tuần 6"
date: 2026-10-19
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### MỤC TIÊU TUẦN 6:
- Thực hành caching in-memory với Amazon ElastiCache for Redis.
- Dựng ứng dụng web sẵn sàng cao với Auto Scaling Group, load balancer và RDS.
- Bắt đầu serverless với Lambda để tối ưu chi phí EC2.
- Viết các quyết định về nguồn tin tức và Bedrock (ADR-0003, ADR-0004) và chuẩn bị benchmark cho chatbot.

### CÁC CÔNG VIỆC TRONG TUẦN 6:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | **- Thực hành lab: Amazon ElastiCache for Redis (phần 1)**<br>+ Chuẩn bị: tạo access key, cài và cấu hình AWS CLI<br>+ Tạo subnet group (Console và AWS CLI)<br>+ Tạo cluster ở chế độ cluster mode disabled và enabled<br>+ Cấp quyền truy cập và kết nối tới các node<br>+ Lấy endpoint bằng AWS CLI, xóa cluster<br>**- Dự án nhóm: cập nhật ADR-0003 và ADR-0004 cho MVP**<br>+ ADR-0003: nguồn tin tức và tín hiệu<br>+ ADR-0004: Region và quyền truy cập Bedrock (Nova 2 Lite cho câu hỏi đơn giản, Claude Haiku 4.5 khi cần tool) | 19/10/2026 | 19/10/2026 | [Amazon ElastiCache - Redis](https://000061.awsfcaj.com/vi/)<br>[Bedrock model access](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html) |
| **3** | **- Thực hành lab: Amazon ElastiCache for Redis (phần 2)**<br>+ Dùng AWS SDK để đọc/ghi dữ liệu trên ElastiCache:<br>* Tạo cluster (cluster mode disabled và enabled), kiểm tra user group<br>* Kết nối tới cluster<br>* Set/Get chuỗi và hash<br>* Publish/Subscribe và đọc/ghi stream<br>+ Dọn dẹp tài nguyên<br>**- Dự án nhóm: thống nhất kế hoạch gán nhãn tin tức với các ADR của MVP**<br>+ Cập nhật prompt gán nhãn tin tức cho khớp với các ADR | 20/10/2026 | 20/10/2026 | [Amazon ElastiCache - Redis](https://000061.awsfcaj.com/vi/) |
| **4** | **- Thực hành lab: Triển khai ứng dụng FCJ Management với Auto Scaling Group**<br>+ Chuẩn bị: hạ tầng mạng, EC2, RDS và dữ liệu mẫu, web server, metric cho predictive scaling<br>+ Tạo launch template<br>+ Thiết lập load balancer: target group và Application Load Balancer<br>+ Tạo Auto Scaling Group<br>+ Kiểm tra scaling thủ công, theo lịch, động (dynamic) và đọc metric predictive scaling<br>+ Dọn dẹp tài nguyên<br>**- Dự án nhóm: hoàn thiện ADR-0003**<br>+ Ghi rõ điều khoản sử dụng của từng nguồn tin | 21/10/2026 | 21/10/2026 | [Triển khai ứng dụng FCAJ Management với Auto Scaling Group](https://000006.awsfcaj.com/vi/) |
| **5** | **- Thực hành lab: Triển khai WordPress trên AWS**<br>+ Chuẩn bị: VPC và subnet, security group cho EC2 và database, EC2 và RDS instance<br>+ Cài WordPress trên EC2<br>+ Auto Scaling cho WordPress: tạo AMI từ web server, launch template, target group, load balancer, Auto Scaling Group<br>+ Sao lưu và khôi phục database bằng DB snapshot<br>+ Đặt CloudFront trước web server<br>+ Dọn dẹp tài nguyên<br>**- Dự án nhóm: benchmark gán nhãn tin tức với Amazon Nova**<br>+ Prompt gán nhãn tin tức v2, JSON schema cho đầu ra, giới hạn max_tokens<br>+ Bộ case benchmark và báo cáo lần chạy đầu tiên | 22/10/2026 | 22/10/2026 | [Triển khai Wordpress trên AWS Cloud](https://000021.awsfcaj.com/vi/)<br>[Amazon Nova](https://docs.aws.amazon.com/nova/latest/userguide/what-is-nova.html) |
| **6** | **- Thực hành lab: Tối ưu chi phí EC2 với AWS Lambda**<br>+ Chuẩn bị: VPC, security group, EC2 instance, Slack incoming webhook<br>+ Gắn tag cho instance<br>+ Tạo IAM role cho Lambda<br>+ Tạo Lambda function dừng và khởi động instance<br>+ Kiểm tra kết quả và dọn dẹp tài nguyên<br>**- Dự án nhóm: kịch bản demo MVP và benchmark chatbot**<br>+ Kiểm tra thủ công bằng chứng của kết quả gán nhãn<br>+ Kịch bản demo MVP cho ngày 25/10<br>+ Đề xuất benchmark chatbot: system prompt, bộ câu hỏi, fixture<br>+ Chạy benchmark lần đầu, sau đó viết script benchmark theo nhánh có kiểm tra tự động (hoàn thành trong cuối tuần) | 23/10/2026 | 23/10/2026 | [Optimize EC2 cost with Lambda](https://000022.awsfcaj.com/vi/)<br>[Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html) |

### KẾT QUẢ ĐẠT ĐƯỢC TRONG TUẦN 6:
1. Tạo ElastiCache Redis cluster (cluster mode disabled và enabled), thao tác qua CLI và SDK (string, hash, pub/sub, stream).
2. Triển khai ứng dụng FCJ Management và WordPress với Auto Scaling Group, ALB, snapshot RDS và CloudFront.
3. Dùng Lambda để dừng/khởi động EC2 có gắn tag theo lịch và gửi thông báo qua Slack.
4. Cập nhật ADR-0003, ADR-0004 cho MVP, chạy benchmark gán nhãn tin tức đầu tiên với Amazon Nova, viết đề xuất và script benchmark cho chatbot.
