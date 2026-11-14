---
title: "Worklog Tuần 9"
date: 2026-11-09
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### MỤC TIÊU TUẦN 9:
- Giám sát mạng và metric ứng dụng bằng VPC Flow Logs, Grafana và CloudWatch.
- Kiểm tra link trích dẫn trong câu trả lời của chatbot và hoàn thiện prompt v4.
- Chuẩn bị kịch bản hỏi đáp, chương báo cáo và slide cho phần chatbot trong buổi bảo vệ.

### CÁC CÔNG VIỆC TRONG TUẦN 9:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | **- Dự án nhóm: kiểm tra link trích dẫn**<br>+ Tổng kết buổi chạy thử chatbot nội bộ ngày 08/11<br>+ Đối chiếu link trích dẫn với các tin của get_news (ghi log trong Lambda, đánh trượt trong benchmark)<br>+ Chốt prompt Haiku v4 sau lần chạy dev lại, đặt ngưỡng riêng cho từng bộ | 09/11/2026 | 09/11/2026 | [Lambda logging](https://docs.aws.amazon.com/lambda/latest/dg/python-logging.html) |
| **3** | **- Thực hành lab: VPC Flow Logs và Grafana trên AWS**<br>+ VPC Flow Logs: bật flow log và giám sát hạ tầng mạng, dọn dẹp tài nguyên<br>+ Grafana: tạo VPC, subnet, security group và EC2, tạo IAM user và role, cài Grafana, dựng dashboard giám sát, dọn dẹp tài nguyên<br>**- Dự án nhóm: quy tắc benchmark cho v4**<br>+ Đóng phương án gọi lại router trong đề xuất router<br>+ Yêu cầu mọi câu hỏi về chính sách phải đạt trong bộ chính thức<br>+ Review PR job tính độ chính xác cuốn chiếu (#89); Lambda chatbot có log trích dẫn được deploy trong ngày | 10/11/2026 | 10/11/2026 | [Giám sát hạ tầng mạng với VPC Flow Log](https://000074.awsfcaj.com/vi/)<br>[Bắt đầu với Grafana trên AWS](https://000029.awsfcaj.com/vi/) |
| **4** | **- Thực hành lab: Amazon CloudWatch Workshop**<br>+ CloudWatch metric: xem metric, search expression, math expression, dynamic label<br>+ CloudWatch Logs, Logs Insights và metric filter<br>+ CloudWatch alarm<br>+ CloudWatch dashboard<br>+ Dọn dẹp tài nguyên<br>**- Dự án nhóm: kịch bản hỏi đáp cho phần bảo vệ chatbot**<br>+ Kịch bản hỏi đáp và kiểm tra câu hỏi demo không trùng với các bộ benchmark | 11/11/2026 | 11/11/2026 | [AWS CloudWatch Workshop](https://000036.awsfcaj.com/vi/) |
| **5** | **- Dự án nhóm: cập nhật chương chatbot với kết quả chính thức của v4**<br>+ Bổ sung kết quả chính thức của prompt v4 vào chương chatbot<br>+ Chạy lại các câu hỏi dự phòng của kịch bản hỏi đáp và cập nhật | 12/11/2026 | 12/11/2026 |  |
| **6** | **- Dự án nhóm: slide chatbot cho buổi thuyết trình cuối kỳ**<br>+ Slide phần chatbot cho buổi thuyết trình cuối kỳ<br>+ Cập nhật bộ tin demo với báo cáo STEO và báo cáo tuần mới nhất của EIA | 13/11/2026 | 13/11/2026 |  |

### KẾT QUẢ ĐẠT ĐƯỢC TRONG TUẦN 9:
1. Giám sát lưu lượng mạng bằng VPC Flow Logs, dựng dashboard Grafana và CloudWatch, alarm và truy vấn Logs Insights.
2. Thêm kiểm tra link trích dẫn cho chatbot và benchmark, chốt prompt Haiku v4.
3. Hoàn thành kịch bản hỏi đáp, chương chatbot với kết quả chính thức của v4 và slide chatbot cho buổi thuyết trình cuối kỳ.
