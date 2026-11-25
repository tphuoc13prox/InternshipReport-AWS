---
title: "Thu thập dữ liệu giá"
date: 2026-11-20
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---
#### Nguồn dữ liệu (ADR-0005)

Mỗi chuỗi lấy từ **một nguồn duy nhất** cho cả lịch sử lẫn cập nhật hằng ngày, để không bị lệch khi nối hai nguồn.

| Tài sản | Chuỗi target | Lịch giao dịch | Đối chiếu |
|---|---|---|---|
| Vàng | GLD (quỹ ETF giữ vàng vật chất) | NYSE | Mức giá vàng của NBP |
| Bạc | SLV (quỹ ETF giữ bạc vật chất) | NYSE | — |
| Dầu Brent | BZ=F | CME | EIA API v2 (RBRTE) |
| Dầu WTI | CL=F | CME | EIA API v2 (RWTC) |
| USD/VND | VND=X | — | Chỉ dùng quy đổi hiển thị, không làm đặc trưng |

Cả năm chuỗi đều tải qua thư viện `yfinance`. README của yfinance ghi rõ API của Yahoo Finance chỉ dành cho mục đích cá nhân, và nhóm không có giấy phép của Yahoo. Nhóm coi đây là rủi ro đã biết và giảm phạm vi sử dụng như sau:

- web luôn ghi nguồn dữ liệu và chỉ mở sau khi đăng nhập;
- web không có tính năng tải dữ liệu giá;
- hiển thị giá vàng/bạc tắt được bằng cờ build `VITE_SHOW_METAL_PRICES=false`.

#### Chạy trên máy

```bash
# nạp lịch sử từ 2012 vào thư mục data/lake (đã nằm trong .gitignore)
python -m src.data.ingest_prices --start 2012-01-01 --dest local

# chạy thường lệ: bỏ --start thì chỉ tải lại 5 phiên gần nhất
python -m src.data.ingest_prices --dest local
```

Lambda ingest (`src/data/lambda_ingest.py`) gọi cùng hàm `run()` với đích S3 và nhận ba loại event:

| Event | Hành vi |
|---|---|
| `{}` | Chạy thường lệ: tải lại 5 phiên gần nhất, ghi `raw/` và `clean/` |
| `{"start": "2012-01-01"}` | Nạp lịch sử lần đầu khi bucket còn trống |
| `{"dry_run": true}` | Tải 10 phiên gần nhất của 5 mã chính, **không ghi gì**, trả về số dòng và lỗi của từng mã. Dùng để kiểm tra Yahoo có chặn IP của AWS không |

#### Quy tắc làm sạch

- `pandas_market_calendars` xác định phiên hợp lệ: lịch NYSE cho GLD và SLV, lịch CME cho dầu.
- **Bỏ thanh của ngày hiện tại** theo giờ New York, vì phiên đó chưa đóng cửa.
- Mỗi lần chạy **tải lại 5 phiên gần nhất** và ghi đè theo khóa, để sửa dữ liệu mà nhà cung cấp cập nhật muộn. Chạy lại trong cùng ngày cho cùng kết quả.
- Bảng clean ở dạng dài, mỗi dòng có cột `flag`. Dòng bị loại được ghi vào `rejected.parquet` kèm lý do, không bị xóa im lặng.
- **Không forward-fill target.** Giá ≤ 0 được giữ, nhưng return là NaN. Ví dụ: WTI ngày 20/4/2020 có giá âm.
- **Lỗ hổng đã biết:** Yahoo thiếu phiên 5/12/2018 của BZ=F và CL=F. Hôm đó NYSE đóng cửa vì quốc tang, nhưng CME năng lượng vẫn giao dịch. Nhóm không lấp phiên này, nên các cửa sổ chứa ngày đó được đánh dấu `invalid_input_window`.

#### Bố cục trên S3

```text
s3://cpaws-dev-lake/
├── raw/yfinance/dt=<ngày>/{GLD,SLV,CL_F,BZ_F,VND_X,...}.csv.gz
├── raw/eia/dt=<ngày>/ , raw/nbp/dt=<ngày>/
├── clean/prices/dt=<ngày>/{prices.parquet, rejected.parquet, quality.json}
├── clean/reference_prices/ , clean/display_fx/ , clean/display_factors/
├── models/        # artifact mô hình thử nghiệm
└── eval/          # kết quả validation, test, shadow (chỉ ghi thêm, không ghi đè)
```

Mỗi ngày chạy tạo một phân vùng `dt=` mới. Predict đọc phân vùng mới nhất. Bucket bật versioning nên có thể dựng lại đúng snapshot đã dùng cho một lần đánh giá.

#### Kiểm tra

```bash
pytest -q tests/test_data_ingest.py tests/test_lambda_ingest.py tests/test_data_clean_quality.py
```

Khi một nguồn chính lỗi (ví dụ Yahoo trả lỗi 429 cho một mã), các tài sản khác vẫn được ghi. Tài sản lỗi giữ dữ liệu cũ, có cờ `stale_source`. Lambda ghi log `INGEST_SOURCE_FAILED` rồi báo lỗi để alarm gửi email. Lần chạy thành công kế tiếp tự tải bù phần còn thiếu.

![S3 clean/prices](/images/5-Workshop/5.3.1-s3-clean-prices.png)

*Thư mục `clean/prices/` trong bucket `cpaws-dev-lake`: mỗi lượt ingest tạo một phân vùng `dt=<ngày>`, phân vùng đầu tiên là `dt=2026-10-15`.*
