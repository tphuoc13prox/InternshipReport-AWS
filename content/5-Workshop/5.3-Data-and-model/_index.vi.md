---
title: "Dữ liệu và mô hình"
date: 2026-11-20
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---
#### Tổng quan

Phần này xây hai khối logic, chạy giống nhau trên máy và trong Lambda:

1. [Thu thập dữ liệu giá](5.3.1-ingest-prices/): tải giá theo lịch giao dịch, làm sạch, gắn cờ, ghi snapshot lên S3.
2. [Mốc dự báo phân vị](5.3.2-baseline-forecast/): tính target log return, mốc 252 phiên, chuẩn hóa phân vị, xác suất tăng, rồi đánh giá bằng pinball loss.

#### Cấu trúc thư mục

```text
src/
├── data/        # ingest_prices.py (CLI + run()), lambda_ingest.py, calendars.py, clean.py
├── features/    # build.py: đặc trưng dùng chung cho train và predict
├── forecast/    # baselines.py, returns.py, predict.py, lambda_predict.py, forecast_record.py
├── eval/        # metrics.py, walk_forward.py, rolling_job.py, run_test2020.py
└── chatbot/     # chat.py, tools.py, citations.py, quota.py, router.py
infra/
├── app.py       # điểm vào CDK, tham số -c và giá trị mặc định
├── stacks/      # 5 stack: data, serving, auth, api, ops
└── lambda/      # handler của forecasts-api, accuracy-api, chat-api
docker/          # Dockerfile của image ingest và predict
web/             # React + TypeScript + Vite
tests/
```

#### Chia dữ liệu

| Giai đoạn | Dùng để |
|---|---|
| 2012–2013 | Khởi động, để đủ 252 phiên lịch sử cho mốc |
| 2014–2018 | Huấn luyện |
| 2019 | Validation: chọn mô hình và tham số |
| 2020 → nay | Test: **chỉ chạy một lần** sau khi khóa mọi tham số |

#### Nguyên tắc chống rò rỉ dữ liệu

- Tại thời điểm dự báo t, chỉ dùng thông tin **đã công bố trước t**. Dữ liệu phiên D chỉ được coi là có từ 12:00 UTC ngày D+1 (`PUBLISHED_AT_RULES`). Lambda chạy lúc 12:15 UTC, nên production dùng đúng mốc thông tin như backtest.
- Train và predict gọi **cùng một hàm** tạo đặc trưng (`src/features/build.py`).
- Return h phiên bắt đầu tại s chỉ được biết ở phiên s + h, nên mốc tại t chỉ dùng các return **đã kết thúc**.
- Tập test 2020 → nay không được xem trước khi khóa tham số. Nhóm chạy tập này đúng một lần, ngày 16/11/2026.
- CI chạy `tests/test_no_leakage.py` ở mọi PR: nối thêm dữ liệu sau t thì đặc trưng tại t không được thay đổi, và phân rã CEEMDAN trên cả chuỗi phải bị phát hiện là rò rỉ.
