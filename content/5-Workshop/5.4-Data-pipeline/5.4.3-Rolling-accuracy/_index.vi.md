---
title: "Đo độ tin cậy"
date: 2025-09-09
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

Một trang dự báo chỉ trung thực khi người xem thấy được các dự báo trước đây đúng sai ra sao. CPAWS làm việc này bằng một job chạy theo lô, chấm điểm chính các dự báo đã phát hành, nên web không phải tính gì khi mở trang.

#### Job làm gì

+ **Lịch:** `cpaws-dev-accuracy-daily`, `cron(45 13 ? * TUE-SAT *)` UTC, tức 20:45 giờ Việt Nam, sau lượt 19:15 là 90 phút.
+ **Đầu vào:** dự báo đã phát hành trong `cpaws-dev-forecasts` (chỉ `Query`), snapshot giá trong `clean/prices/` và file shadow S1a.
+ **Cửa sổ:** `rolling_8w`, gồm các ngày cơ sở trong 56 ngày lịch tính đến ngày chạy. Không bao giờ bù dự báo cũ vào cho đầy cửa sổ.
+ **Chỉ số:** `coverage_80` (tỷ lệ giá thật rơi vào khoảng dự báo 80% danh nghĩa), `width_80`, `sign_accuracy` và `up_rate`, cho từng kỳ hạn trong bảy kỳ hạn (1 đến 7 phiên tới).
+ **Đầu ra:** kết quả ghi vào `cpaws-dev-accuracy` bằng conditional write; bản so sánh champion với S1a ghi vào `eval/shadow_eval/` và không bị ghi đè.

Cặp nào chưa có giá thực tế thì chưa được chấm, nên các ngày cơ sở mới nhất lúc đầu chỉ được tính ở những kỳ hạn ngắn.

#### Ngưỡng hiển thị

Con số tính từ vài ngày trông thì chính xác nhưng ít ý nghĩa. Khi chưa đủ **20 ngày cơ sở khác nhau** được chấm, API trả `data_status = collecting`, còn web để dòng "Đang thu thập (n/20)" ở chỗ lẽ ra là con số. Ngày 12/11 cả bốn tài sản ở mức 9/20; mỗi lượt chạy thêm một ngày cơ sở.

#### Deploy và bật lịch

Nhóm triển khai theo từng bước nhỏ để kiểm được từng phần riêng:

| Ngày | Thay đổi | Stack |
|---|---|---|
| 10/11 | Bảng `cpaws-dev-accuracy` | `CpawsServing` |
| 10/11 | Route `GET /accuracy/{asset}` và Lambda chỉ đọc `cpaws-dev-accuracy-api` | `CpawsApi` |
| 11/11 | Lambda `cpaws-dev-accuracy-job` (image `e31c928`), lịch ở trạng thái **tắt** | `CpawsData` |
| 11/11 | Alarm `cpaws-dev-accuracy-job-errors` | `CpawsOps` |
| 11/11 | Dry-run bằng tay, 4 tài sản | — |
| 12/11 | Bật lịch; `cdk diff` chỉ đổi `State` của schedule | `CpawsData` |
| 13/11 | Build web với `VITE_ACCURACY_API=true` | Amplify |

Muốn bật hay tắt lịch thì deploy lại `CpawsData`; mặc định là bật:

```bash
cdk deploy CpawsData -c accuracy_schedule=off --profile cpaws-dev
```

Chỉ `State` của schedule thay đổi. Lambda, IAM role và dữ liệu đã ghi giữ nguyên.

#### Kiểm một lượt chạy

```bash
aws logs tail /aws/lambda/cpaws-dev-accuracy-job --since 1h --format short --profile cpaws-dev \
  --filter-pattern "?REPORT ?ACCURACY_DONE ?ACCURACY_FAILED ?ACCURACY_NOT_READY ?WARNING ?ERROR ?Traceback"
aws scheduler get-schedule --name cpaws-dev-accuracy-daily --profile cpaws-dev \
  --query "[State,ScheduleExpression]" --output text
```

<!-- ẢNH: CloudWatch Logs, log job accuracy có ACCURACY_DONE cho bốn tài sản -->
[ẢNH: CloudWatch Logs – job accuracy có `ACCURACY_DONE`]

| Lượt | Chế độ | Duration | Bộ nhớ tối đa | Kết quả |
|---|---|---:|---:|---|
| 11/11 20:58 | Dry-run, gọi tay | 23,7 s | 290 / 1.024 MB | 4 tài sản, không lỗi |
| 12/11 20:45 | Theo lịch | 30,3 s | 297 / 1.024 MB | 4 tài sản `COMPLETED`, 9 ngày cơ sở, alarm OK |

Job còn ghi cảnh báo `prob_up_mismatch` khi xác suất tăng đã lưu lệch với giá trị tính lại từ các phân vị. Ngày 12/11 độ lệch lớn nhất dưới 1e-5, do làm tròn 6 chữ số khi ghi DynamoDB, nên nhóm giữ ngưỡng và không sửa job.
