---
title: "Các bài blogs đã đăng"
date: 2025-09-09
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Tối thiểu 3 bài blog kỹ thuật đã đăng lên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/).

### [Blog 1 - AgentCore Memory: Vì sao agent vừa được kể xong đã "quên"?](3.1-Blog1/)

Bài tự viết, lấy ý tưởng từ slide "AgentCore Memory" tại sự kiện FCAJ x FPTU: Cloud Mastery 1. Bài giải thích hai tầng của Amazon Bedrock AgentCore Memory (short-term ghi đồng bộ dạng event, long-term được trích xuất bất đồng bộ thành memory record) qua 5 luồng dữ liệu được đánh số, chỉ ra vì sao agent chưa truy xuất được sở thích người dùng vừa nói, và đề xuất cách đọc cả hai tầng (`ListEvents` + `RetrieveMemoryRecords`), nạp trực tiếp bằng `BatchCreateMemoryRecords`/`IngestData`, kèm checklist khi agent "không nhớ".

Phan Tấn Phước đăng ngày 14/11/2026: [bài gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2337675363664089/)

### [Blog 2 - Tool use với Bedrock Converse API: Model gọi đúng tool rồi, sao vẫn trả lời sai?](3.2-Blog2/)

Bài tự viết, rút ra từ kinh nghiệm làm chatbot trên Amazon Bedrock, minh họa bằng ví dụ chatbot thời tiết. Bài giải thích vòng lặp tool use 5 bước của Converse API (`toolUse` → chạy tool → `toolResult` → lặp tới `end_turn`), các "phanh" cần có khi chạy thật (giới hạn số vòng, hạn chót dưới timeout của API Gateway, xử lý mọi `stopReason`), cách báo lỗi tool có cấu trúc, và vì sao phải thiết kế output của tool cho model dễ đọc đúng, kèm checklist.

Phan Tấn Phước đăng ngày 16/11/2026: [bài gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2339395856825373/)

### [Blog 3 - Prompt caching cho chatbot có tool use: Đặt cachePoint đúng chỗ để giảm tới ~90% chi phí phần prompt lặp lại trên Amazon Bedrock](3.3-Blog3/)

Bài tự viết, tổng hợp từ tài liệu AWS, các con số là ví dụ minh họa. Bài giải thích prompt caching trong Converse API (`cachePoint` ở tools, system, messages; giá ghi và đọc cache; TTL), ước tính mức tiết kiệm cho phần lặp lại trong vòng lặp tool use, và đề xuất cách sắp xếp prompt để cache thật sự trúng (tĩnh trước, động sau; danh sách tool ổn định; mốc cache di động ở cuối message mới nhất), kèm các bẫy (ngưỡng token tối thiểu, không đảm bảo trúng) và cách đo tỉ lệ trúng.

Phan Tấn Phước đăng ngày 17/11/2026: [bài gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2340410270057265/)
