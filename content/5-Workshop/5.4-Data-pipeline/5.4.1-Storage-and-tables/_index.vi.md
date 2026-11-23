---
title: "Lưu trữ và bảng dữ liệu"
date: 2025-09-09
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

#### Bốn bảng DynamoDB (`CpawsServing`)

`CpawsServing` tạo mọi bảng mà hệ thống đọc khi có request. Tất cả dùng chế độ on-demand vì lưu lượng thấp và không đều.

| Bảng | Khóa | Ai ghi | Ai đọc | Removal policy |
|---|---|---|---|---|
| `cpaws-dev-forecasts` | `asset` + `base_date` | Lambda predict (`PutItem` có điều kiện) | forecasts-api, `get_forecast` của chat, job accuracy | `RETAIN` |
| `cpaws-dev-news-labels` | `asset` + `sk` (`<published_at>#<article_id>`) | Collector tin (dự kiến) | `get_news` của chat | `RETAIN` |
| `cpaws-dev-chat-usage` | `pk` + `sk`, TTL theo `expires_at` | chat-api (bộ đếm lượt) | chat-api | `DESTROY` |
| `cpaws-dev-accuracy` | `PK` + `SK` | Job accuracy (ghi có điều kiện) | accuracy-api | `RETAIN` |

`chat-usage` chỉ lưu bộ đếm, không bao giờ lưu câu hỏi hay câu trả lời, nên xóa cùng stack cũng không sao. Bảng dự báo, bảng tin và bảng độ tin cậy được giữ lại khi xóa stack vì chứa lịch sử đã phát hành.

Deploy:

```bash
cdk diff CpawsServing --profile cpaws-dev
cdk deploy CpawsServing --profile cpaws-dev
```

Kiểm tra kết quả:

```bash
aws dynamodb list-tables --profile cpaws-dev --query "TableNames[?starts_with(@, 'cpaws-dev')]"
```

<!-- ẢNH: DynamoDB console, bốn bảng cpaws-dev-* ở chế độ On-demand -->
[ẢNH: DynamoDB console – bốn bảng `cpaws-dev-*`, on-demand]

#### Data lake S3 (do `CpawsData` tạo)

Bucket `cpaws-dev-lake` được tạo ở bước sau cùng với các Lambda, nhưng cấu hình của nó ảnh hưởng tới cả pipeline:

+ chặn truy cập công khai, bắt buộc SSL;
+ bật versioning, bản cũ hết hạn sau 30 ngày;
+ `RETAIN`, xóa stack cũng không xóa dữ liệu.

Các prefix hệ thống dùng:

| Prefix | Nội dung |
|---|---|
| `raw/` | File tải nguyên trạng từ nguồn |
| `clean/prices/dt=<ngày>/` | Mỗi lượt chạy một snapshot giá đã làm sạch |
| `eval/shadow/s1a-l094/dt=<ngày>/` | Dự báo shadow của challenger S1a |
| `eval/shadow_eval/` | So sánh champion với S1a do job accuracy ghi |
| `serving/price-history` | Lịch sử giá ngắn, forecasts-api đọc để vẽ biểu đồ 2 năm (từ v1.0.0) |
