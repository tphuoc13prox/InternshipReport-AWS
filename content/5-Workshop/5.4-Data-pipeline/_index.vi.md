---
title: "Pipeline dữ liệu"
date: 2025-09-09
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Phần này dựng phần back end chạy không cần người dùng: các bảng, data lake S3, hai image container, chuỗi ingest → predict hằng ngày và job đo độ tin cậy. Đây là luồng 1 và 2 trên [sơ đồ kiến trúc](../5.3-architecture/).

Làm việc trong thư mục `infra/` với virtual environment đang bật. Nhóm có vài quy tắc cho mọi lần deploy, nên áp dụng theo:

+ Chạy `cdk diff <Stack>` và đọc kỹ trước khi `cdk deploy <Stack>`.
+ Mỗi lệnh deploy một stack, mỗi lúc chỉ một người deploy.
+ Không deploy `CpawsData` trong khung 19:05–19:35 giờ Việt Nam để không đụng lượt chạy 19:15.
+ Gắn tag image bằng SHA ngắn của commit merge, không bao giờ dùng `latest`.

Thứ tự khi một thay đổi động tới nhiều stack: `CpawsServing` (bảng) → `CpawsData` (Lambda, lịch chạy) → `CpawsOps` (alarm) → `CpawsApi` (route). Tức là tạo tài nguyên trước, rồi mới tạo thứ đọc tài nguyên đó. `CpawsAuth` chỉ cần có trước `CpawsApi`.

#### Nội dung

1. [Lưu trữ và bảng dữ liệu](5.4.1-storage-and-tables/)
2. [Ingest và predict](5.4.2-ingest-and-predict/)
3. [Đo độ tin cậy](5.4.3-rolling-accuracy/)
