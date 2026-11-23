---
title: "Các bài blogs đã đăng"
date: 2025-09-09
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

5 bài blog kỹ thuật của nhóm đã đăng lên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/), mỗi thành viên một bài.

### [Blog 1 - Prompt caching cho chatbot có tool use: Đặt cachePoint đúng chỗ để giảm tới ~90% chi phí phần prompt lặp lại trên Amazon Bedrock](3.1-Blog1/)

Bài tổng hợp từ tài liệu AWS, các con số là ví dụ minh họa. Bài giải thích prompt caching trong Converse API (`cachePoint` ở tools, system, messages; giá ghi và đọc cache; TTL), ước tính mức tiết kiệm cho phần lặp lại trong vòng lặp tool use, và đề xuất cách sắp xếp prompt để cache thật sự trúng (tĩnh trước, động sau; danh sách tool ổn định; mốc cache di động ở cuối message mới nhất), kèm các bẫy cần biết và cách đo tỉ lệ trúng.

Phan Tấn Phước đăng ngày 17/11/2026: [bài gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2340410270057265/)

### [Blog 2 - Khách hàng đặt hàng thành công nhưng hệ thống kho không nhận được event: Transactional Outbox giải quyết thế nào trên AWS?](3.2-Blog2/)

Bài giải thích Dual Write Problem trong kiến trúc microservices và cách Transactional Outbox Pattern giải quyết: ghi đơn hàng và outbox event trong cùng một DynamoDB transaction (`TransactWriteItems`), rồi dùng DynamoDB Streams, EventBridge Pipes, EventBridge và SQS để phát event bất đồng bộ. Bài cũng nói vì sao consumer vẫn cần idempotency và khi nào nên dùng kiến trúc này.

Huỳnh Ngọc Tuấn đăng ngày 16/11/2026: [bài gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338993156865643/)

### [Blog 3 - Mô hình chỉ chạy vài giây mỗi ngày: Có cần đến SageMaker Endpoint không?](3.3-Blog3/)

Bài so sánh bốn cách chạy inference trên SageMaker và chỉ ra vì sao một job dự báo nhỏ chạy mỗi đêm hợp với Lambda container image hơn real-time endpoint. Bài đi qua pipeline 7 bước (ECR, EventBridge Scheduler, Lambda destinations, S3, DynamoDB, CloudWatch), các bẫy gặp khi chạy thử (`awslambdaric`, `WORKDIR`, gọi đồng bộ và bất đồng bộ), chi phí so với free tier của Lambda, và bảng gợi ý khi nào chọn gì.

Trịnh Tiến Khải đăng ngày 15/11/2026: [bài gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338607513570874/)

### [Blog 4 - Từ SageMaker đến thiết bị: Triển khai mô hình ML xuống edge bằng AWS IoT Greengrass V2](3.4-Blog4/)

Bài tìm hiểu từ AWS IoT Greengrass V2 Developer Guide. Bài giải thích vì sao chạy suy luận ngay trên thiết bị, các khái niệm nền (core device, component, deployment, nucleus), cách chia model/runtime/inference component, điều kiện với bucket S3 và thiết bị, cách tùy biến component mẫu, và các điểm cần lưu ý (Edge Manager đã ngừng, phiên bản nucleus, runtime ghim phiên bản).

Nguyễn Tiến Long đăng ngày 21/11/2026: [bài gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2344828116282147/)

### [Blog 5 - Mỗi tối chạy một việc rồi xử lý tiếp: Có cần một server chạy 24/7 hay Step Functions không?](3.5-Blog5/)

Bài hướng dẫn thay một máy chủ chạy cron bằng EventBridge Scheduler, Lambda (dùng container image khi thư viện nặng) và Lambda Destinations để nối hai bước, kèm CloudWatch Alarm để giám sát. Bài nói về retry policy của Scheduler và Lambda, IAM role tối thiểu quyền để tránh lỗi confused deputy, và khi nào nên chuyển sang Step Functions.

Châu Trần Mạnh Tiến đăng ngày 15/11/2026: [bài gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338839130214379/)
