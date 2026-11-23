---
title: "Bootstrap và stack dữ liệu"
date: 2026-11-20
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---
#### Bootstrap CDK

Chạy một lần cho mỗi cặp tài khoản và region:

```bash
cd infra
pip install -r requirements.txt
cdk bootstrap aws://<ACCOUNT_ID>/ap-southeast-1 --profile cpaws-dev
```

#### Stack `CpawsData` gồm những gì

| Tài nguyên | Cấu hình |
|---|---|
| S3 `cpaws-dev-lake` | Chặn public, bắt buộc SSL, mã hóa SSE-S3, bật versioning, phiên bản cũ giữ 30 ngày, `RemovalPolicy.RETAIN` |
| ECR `cpaws-dev-ingest`, `cpaws-dev-predict` | Quét image khi push, chỉ giữ 5 image mới nhất |
| Lambda `cpaws-dev-ingest` | Container x86_64, 1.024 MB, timeout 300 giây, retry bất đồng bộ 1 lần |
| Lambda `cpaws-dev-predict` | Container x86_64, 1.024 MB, timeout 120 giây, retry 0 |
| Lambda `cpaws-dev-accuracy-job` | Dùng image predict với handler riêng, 1.024 MB, 300 giây, retry 0 |
| EventBridge Scheduler | Lịch ingest 12:15 UTC và lịch job accuracy 13:45 UTC, thứ Ba–thứ Bảy |

Nhóm chọn **Lambda container image** thay cho file zip vì thư viện của ingest (pandas, pyarrow, numpy, yfinance) vượt giới hạn 250 MB của gói zip. Image ingest khi giải nén khoảng 0,9 GB, còn xa giới hạn 10 GB của Lambda.

Base image là `public.ecr.aws/lambda/python:3.12`, **ghim theo digest** trong Dockerfile để build lại ra đúng image cũ. Không dùng bản 3.11 vì base 3.11 chạy Amazon Linux 2 (glibc 2.26), không cài được numpy, pandas và pyarrow ở các phiên bản đã ghim.

#### Build và đẩy image

Tag image bằng SHA ngắn của commit đã merge vào `developer`, không dùng `latest`:

```bash
TAG=$(git rev-parse --short HEAD)
REPO=<ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com
aws ecr get-login-password --profile cpaws-dev | docker login --username AWS --password-stdin $REPO

# --provenance=false để Lambda nhận được image
docker buildx build --platform linux/amd64 --provenance=false \
  -f docker/ingest/Dockerfile -t $REPO/cpaws-dev-ingest:$TAG --push .
docker buildx build --platform linux/amd64 --provenance=false \
  -f docker/predict/Dockerfile -t $REPO/cpaws-dev-predict:$TAG --push .
```

`.dockerignore` chỉ cho vào image phần code cần thiết, không có `data/`, `tests/` hay `.env`.

#### Deploy

Lambda container chỉ tạo được khi image đã có trong ECR. Nhóm deploy theo từng bước:

1. Tạo bucket và repo ECR trước.
2. Push image ingest, rồi thêm Lambda ingest.
3. Với predict, stack có cờ `-c predict_repo_only=true`. Lần deploy đầu bật cờ này để chỉ tạo repo. Sau khi push image predict thì deploy lại **không** có cờ.

{{% notice warning %}}
Đừng dùng `predict_repo_only=true` ở các lần deploy sau: lần deploy đó sẽ xóa Lambda predict và destination của ingest.
{{% /notice %}}

Các lần deploy thường lệ:

```bash
cdk diff CpawsData --profile cpaws-dev
cdk deploy CpawsData --profile cpaws-dev
```

Muốn thử image mới mà chưa sửa `app.py`, truyền tag bằng `-c ingest_image_tag=$TAG` hoặc `-c predict_image_tag=$TAG`. Lần deploy sau không truyền thì tag quay về giá trị trong `app.py`.

#### Kiểm tra sau deploy

Ngay sau lần deploy đầu, gọi ingest ở chế độ `dry_run` để biết Yahoo có chặn IP của AWS không:

```bash
aws lambda invoke --function-name cpaws-dev-ingest --profile cpaws-dev \
  --cli-binary-format raw-in-base64-out --payload '{"dry_run": true}' out.json
cat out.json   # mỗi mã có rows khoảng 10 và error = null
```

Nếu `dry_run` ổn, nạp lịch sử bằng event `{"start": "2012-01-01"}` (gọi `--invocation-type Event`), rồi kiểm tra trên S3 đã có `clean/prices/dt=<ngày>/`.

<!-- ẢNH: Console ECR repo cpaws-dev-predict với các tag SHA (che Account ID trong URI) -->
<!-- ẢNH: Console Lambda cpaws-dev-ingest: Image URI, memory, timeout (che Account ID) -->
