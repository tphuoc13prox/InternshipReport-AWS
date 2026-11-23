---
title: "Ingest và predict"
date: 2025-09-09
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

#### `CpawsData` tạo những gì

| Tài nguyên | Cấu hình |
|---|---|
| S3 `cpaws-dev-lake` | Xem [5.4.1](../5.4.1-storage-and-tables/) |
| ECR `cpaws-dev-ingest`, `cpaws-dev-predict` | Quét khi push, giữ 5 image mới nhất |
| Lambda `cpaws-dev-ingest` | Container, x86_64, 1.024 MB, 300 giây, retry bất đồng bộ 1 lần |
| Lambda `cpaws-dev-predict` | Container, x86_64, 1.024 MB, 120 giây, retry 0 |
| Lambda `cpaws-dev-accuracy-job` | Cùng image với predict, lệnh riêng, 1.024 MB, 300 giây, retry 0 ([5.4.3](../5.4.3-rolling-accuracy/)) |
| EventBridge Scheduler `cpaws-dev-ingest-daily` | `cron(15 12 ? * TUE-SAT *)` UTC, retry 0 |

Không Lambda nào chạy trong VPC, nên không cần NAT Gateway.

#### Bước 1 – Build image

Dockerfile nằm trong `docker/`, base image `public.ecr.aws/lambda/python:3.12` được ghim theo digest. Build từ thư mục gốc của repo:

```bash
docker buildx build --platform linux/amd64 --provenance=false \
  -f docker/ingest/Dockerfile -t cpaws-ingest:dev .
docker buildx build --platform linux/amd64 --provenance=false \
  -f docker/predict/Dockerfile -t cpaws-predict:dev .
```

Cờ `--provenance=false` là bắt buộc: thiếu nó, buildx tạo manifest list mà Lambda không nhận.

#### Bước 2 – Tạo repo predict rồi push image

Lambda dạng container chỉ tạo được khi image đã có sẵn trong ECR. Ở lần deploy đầu tiên, chỉ tạo repo predict:

```bash
cdk deploy CpawsData -c predict_repo_only=true --profile cpaws-dev
```

Sau đó đăng nhập ECR và push, gắn tag bằng SHA ngắn của commit (`<ACCOUNT_ID>` là tài khoản của bạn):

```bash
aws ecr get-login-password --region ap-southeast-1 --profile cpaws-dev | \
  docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com
docker tag cpaws-predict:dev <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com/cpaws-dev-predict:<SHA>
docker push <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com/cpaws-dev-predict:<SHA>
```

Làm tương tự cho image ingest với repo `cpaws-dev-ingest`. Rồi điền tag vào `DEFAULT_INGEST_IMAGE_TAG`, `DEFAULT_PREDICT_IMAGE_TAG` và `DEFAULT_ACCURACY_IMAGE_TAG` trong `infra/app.py` (nhóm làm việc này qua pull request).

{{% notice warning %}}
Chỉ dùng `predict_repo_only=true` một lần. Deploy với cờ này ở các lần sau sẽ xóa Lambda predict và destination của ingest.
{{% /notice %}}

#### Bước 3 – Deploy cả stack

```bash
cdk diff CpawsData --profile cpaws-dev
cdk deploy CpawsData --profile cpaws-dev
```

Trong diff, để ý các thay đổi IAM bất thường. Nhóm còn kiểm diff **không** có `AWS::S3::BucketPolicy`, vì bucket policy có một statement thêm tay và CDK sẽ gỡ nó đi.

#### Chuỗi chạy hoạt động thế nào

```
EventBridge Scheduler (19:15 giờ VN, thứ Ba–thứ Bảy)
  └─► Lambda ingest ── ghi raw/ và clean/prices/dt=<ngày>/ lên S3
        └─► OnSuccess hoặc OnFailure ─► Lambda predict
              ├─► cpaws-dev-forecasts   (champion baseline-q252, PutItem có điều kiện)
              └─► eval/shadow/s1a-l094/ (shadow S1a, người dùng không thấy)
```

Ba điểm thiết kế quyết định hệ thống xử lý thế nào khi có lỗi:

+ **Thiếu dữ liệu một phần vẫn ra dự báo.** Ingest lỗi vẫn gọi predict. Predict phát hành cho tài sản nào có phiên mới nhất, bỏ qua tài sản còn lại và ghi `PredictFailed`.
+ **Chạy lại an toàn.** Dự báo đã phát hành cho cùng tài sản và cùng ngày cơ sở không bao giờ bị ghi đè (`PREDICT_ALREADY_PUBLISHED`). Vì vậy predict không retry, còn ingest chỉ retry một lần để không gọi liên tục vào nguồn giá đang lỗi.
+ **Shadow không chặn được champion.** S1a lỗi thì lỗi được ghi log và đếm bằng metric filter; champion vẫn phát hành.

Không cần Step Functions hay SageMaker: hai Lambda nối bằng destination là đủ, còn champion là mốc thống kê, không có artifact mô hình.

#### Bước 4 – Chạy thử không cần đợi 19:15

Gọi ingest bất đồng bộ để destination gọi tiếp predict, giống hệt lúc chạy theo lịch:

```bash
aws lambda invoke --function-name cpaws-dev-ingest --invocation-type Event \
  --payload '{}' --cli-binary-format raw-in-base64-out --profile cpaws-dev out.json
```

Rồi đọc log predict:

```bash
aws logs tail /aws/lambda/cpaws-dev-predict --since 90m --format short --profile cpaws-dev \
  --filter-pattern "?INIT_REPORT ?REPORT ?PREDICT_DONE ?PREDICT_SHADOW ?PREDICT_FAILED ?PREDICT_ALREADY_PUBLISHED ?PREDICT_SKIPPED"
aws s3 ls s3://cpaws-dev-lake/eval/shadow/s1a-l094/dt=YYYY-MM-DD/ --profile cpaws-dev
```

Nếu ngày đó đã phát hành rồi thì sẽ thấy `PREDICT_ALREADY_PUBLISHED` thay vì bản ghi mới. Dòng `PREDICT_DONE` có giá gần nhất và xác suất tăng, nên đừng dán nguyên dòng log vào repo public hay pull request.

<!-- ẢNH: CloudWatch Logs, log predict có dòng PREDICT_DONE và REPORT (che số giá) -->
[ẢNH: CloudWatch Logs – log predict có `PREDICT_DONE` và `REPORT`]

#### Số đo thực tế

Predict chạy tự động lần đầu vào 27/10/2026. Bắt đầu từ 04/11, image `c61abdd` tính cả champion lẫn shadow S1a trong cùng một lượt. Điều kiện để giữ shadow: trong ba lượt đầu, lượt chậm nhất không được chậm hơn mốc 2,9 giây cũ từ 2 giây trở lên:

| Lượt (19:15) | Duration | Bộ nhớ tối đa | File shadow |
|---|---:|---:|---:|
| 04/11 | 3,29 s | — | 4/4 |
| 05/11 | 3,25 s | 260 / 1.024 MB | 4/4 |
| 06/11 | 3,27 s | 258 / 1.024 MB | 4/4 |

Lượt dài nhất chỉ hơn mốc cũ 0,39 giây, nên shadow được giữ.
