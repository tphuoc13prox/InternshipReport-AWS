---
title: "Worklog Tuần 7"
date: 2026-10-26
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:
- Xây dựng back-end serverless với Lambda, S3, DynamoDB, API Gateway và AWS SAM.
- Giám sát và truy vết ứng dụng serverless bằng CloudWatch và X-Ray.
- Viết và được duyệt contract POST /chat, xây dựng Lambda chatbot dùng Bedrock.
- Họp team đánh giá kiến trúc dự án theo AWS Well-Architected.
- Chuẩn bị bộ tin tức gán nhãn thủ công cho tool get_news của chatbot.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 | **- Họp team: đánh giá các tiêu chí well architected và sửa đổi kiến trúc dự án**<br>+ Dự đoán theo lô bằng Lambda container (NumPy) thay vì endpoint SageMaker chạy liên tục<br>**- Dự án nhóm: bộ benchmark v2 và get_forecast v2**<br>+ Bộ câu hỏi v2 có kiểm tra nhãn khoảng dự báo và khóa SHA-256 (chuẩn bị trong cuối tuần)<br>+ get_forecast v2 dùng chung hàm định dạng với Lambda forecast API<br>+ Sửa prompt v2 sau review và các lần chạy dev<br>**- Tham gia Sự kiện 4: Smart Plushies for Children - hoạt động tình nguyện của Amazon (buổi chiều)**<br>+ Chuẩn bị thú bông thông minh cho các em nhỏ được Operation Smile Vietnam hỗ trợ | 26/10/2026 | 26/10/2026 | [AWS Well-Architected](https://aws.amazon.com/architecture/well-architected/)<br>[Bedrock tool use](https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use.html) |
| 3 | **- Thực hành lab: Serverless - Front-end gọi API Gateway**<br>+ Giới thiệu API Gateway và DynamoDB<br>+ Triển khai front-end<br>+ Tạo bảng DynamoDB và các Lambda function ghi, đọc, xóa dữ liệu<br>+ Thiết lập API Gateway: tạo method, cấu hình và bật CORS<br>+ Kiểm tra API bằng Postman và bằng front-end<br>+ Làm lại luồng này với phiên bản cập nhật của lab (000135)<br>+ Dọn dẹp tài nguyên<br>**- Dự án nhóm: contract POST /chat và Lambda chatbot**<br>+ Đề xuất contract POST /chat, sửa theo review và được duyệt<br>+ Phần lõi Lambda chatbot: Claude Haiku trên Bedrock với tool get_news và get_forecast<br>+ Deadline tính từ requestTimeEpoch, giới hạn thời gian gọi Bedrock, quota theo ngày lưu trong DynamoDB<br>+ Thu thập 69 bản tin Fed/EIA ứng viên để gán nhãn | 27/10/2026 | 27/10/2026 | [Serverless - Hướng dẫn viết Front-end gọi API Gateway](https://000079.awsfcaj.com/vi/)<br>[Serverless - Hướng dẫn viết Front-end gọi API Gateway](https://000135.awsfcaj.com/vi/) |
| 4 | **- Thực hành lab: Serverless với Lambda, API Gateway và AWS SAM**<br>+ Chuẩn bị: Cloud9 instance, CodeCommit repository<br>+ Triển khai ứng dụng mẫu: front-end, hạ tầng back-end, dữ liệu chuyến đi, kiểm tra cấu hình<br>+ Triển khai hệ thống thời gian chờ (back-end và front-end)<br>+ Xử lý ảnh chụp trên đường đi: Lambda chroma key và ghép ảnh, hậu xử lý<br>+ Kiểm tra tính năng và dọn dẹp tài nguyên<br>**- Dự án nhóm: gán nhãn tin tức v3**<br>+ Viết bản nháp prompt labeling-v3, mẫu nhãn và script dựng case<br>+ Gán nhãn thủ công 50 bản tin Fed/EIA và 3 trường hợp đặc biệt<br>+ Sửa phần kiểm tra nhãn khoảng dự báo trong benchmark | 28/10/2026 | 28/10/2026 | [Serverless với Lambda, API Gateway và SAM](https://000066.awsfcaj.com/vi/)<br>[Bedrock prompt engineering](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html) |
| 5 | **- Thực hành lab: Giám sát ứng dụng Serverless với CloudWatch và X-Ray**<br>+ Chuẩn bị<br>+ Giám sát bằng CloudWatch: debug bằng CloudWatch Logs, tạo custom metric, tạo CloudWatch alarm<br>+ Truy vết request bằng AWS X-Ray<br>+ Dọn dẹp tài nguyên<br>**- Dự án nhóm: bộ tin demo cho get_news**<br>+ Bộ tin Fed/EIA gán nhãn thủ công và tài liệu hướng dẫn<br>+ Kết quả chạy thật labeling-v3 và bảng kiểm tra bằng chứng thủ công<br>+ Xử lý lỗi: bỏ qua tin không hợp lệ, câu trả lời cố định khi hết số vòng gọi tool, 504 khi model timeout, 400 khi base64 sai | 29/10/2026 | 29/10/2026 | [Serverless - Giám sát Lambda với CloudWatch và X-Ray](https://000085.awsfcaj.com/vi/)<br>[Converse API reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html) |
| 6 | **- Thực hành lab: Serverless - Lambda tương tác với S3 và DynamoDB**<br>+ Xử lý ảnh trên AWS: tạo Lambda function resize ảnh, S3 bucket và IAM policy, kiểm tra function<br>+ Ghi dữ liệu vào Amazon DynamoDB: tạo và quản lý bảng, tạo Lambda function ghi dữ liệu<br>+ Dọn dẹp tài nguyên<br>**- Dự án nhóm: ADR-0013 và kết quả benchmark chatbot v2**<br>+ ADR-0013: dùng nhãn tin tức thủ công đến buổi bảo vệ, không gán nhãn bằng Bedrock trên production<br>+ Kết quả chính thức benchmark chatbot v2 và các quyết định đã thống nhất với nhóm<br>+ Viết bản nháp router dựa trên luật cho chatbot<br>+ Merge nhánh chat API (PR #55); cùng ngày nhóm đã deploy Lambda POST /chat<br>+ Trong cuối tuần: prompt Haiku v3 và các bộ benchmark chỉ dùng cho dev | 30/10/2026 | 30/10/2026 | [Serverless - Tương tác giữa Lambda với S3 và DynamoDB](https://000078.awsfcaj.com/vi/)<br>[Bedrock prompt routing](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-routing.html)<br>[Bedrock Guardrails](https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html) |

### Kết quả đạt được tuần 7:
1. Xây dựng các luồng serverless: resize ảnh với Lambda và S3, ghi DynamoDB, front-end gọi API Gateway có CORS và một ứng dụng AWS SAM hoàn chỉnh.
2. Debug Lambda bằng CloudWatch Logs, custom metric, alarm và truy vết request bằng X-Ray.
3. Contract POST /chat được duyệt; xây dựng Lambda chatbot (Claude Haiku trên Bedrock, tool get_news/get_forecast, timeout, quota DynamoDB).
4. Gán nhãn thủ công các bản tin Fed/EIA cho get_news và ghi quyết định dùng nhãn thủ công vào ADR-0013.
5. Cùng nhóm đánh giá kiến trúc theo AWS Well-Architected, chuyển sang dự đoán theo lô bằng Lambda container.
6. Tham gia hoạt động tình nguyện của Amazon (Sự kiện 4).
