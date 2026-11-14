---
title: "Các bài blog"
date: 2025-09-09
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Tối thiểu 3 bài blog kỹ thuật đã đăng lên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/).

### [Blog 1 - AgentCore Memory: Vì sao agent vừa được kể xong đã "quên"?](3.1-Blog1/)

Bài tự viết, lấy ý tưởng từ slide "AgentCore Memory" tại sự kiện FCAJ x FPTU: Cloud Mastery 1. Bài giải thích hai tầng của Amazon Bedrock AgentCore Memory (short-term ghi đồng bộ dạng event, long-term được trích xuất bất đồng bộ thành memory record) qua 5 luồng dữ liệu được đánh số, chỉ ra vì sao agent chưa truy xuất được sở thích người dùng vừa nói, và đề xuất cách đọc cả hai tầng (`ListEvents` + `RetrieveMemoryRecords`), nạp trực tiếp bằng `BatchCreateMemoryRecords`/`IngestData`, kèm checklist khi agent "không nhớ".

Phan Tấn Phước đăng ngày 14/11/2026: [bài gốc trên AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2337675363664089/)

- Blog 2: [TODO tiêu đề + link]
- Blog 3: [TODO tiêu đề + link]
