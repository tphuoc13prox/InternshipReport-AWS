---
title: "Kiểm thử và giám sát"
date: 2026-11-20
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---
#### CI trên GitHub Actions

Mỗi push và pull request đều chạy:

- **Python:** `ruff check`, `ruff format --check`, `pytest`. Trong đó có các test synth CDK (`tests/test_infra_*.py`) kiểm tra quyền IAM, biến môi trường, lịch Scheduler và alarm.
- **Web:** `npm ci`, `typecheck`, `test`, `build`.
- **Pre-commit:** `gitleaks`, `detect-aws-credentials`, để chặn key và Account ID trước khi đẩy lên repo public.

CI **không deploy**. Nhánh `developer` cần ít nhất 1 approve mới merge được; nhánh `production` chỉ nhận PR release từ `developer`.

Ở bản nộp (nhánh `production`, commit `ac1a31a`, nội dung giống `developer`): **pytest 1.658 test đạt, 2 test bỏ qua** (gồm các test synth CDK), **web 265 test đạt**, `ruff` không báo lỗi và CI xanh ở mọi PR trước khi merge.

#### Kiểm thử sau mỗi lần deploy

| Kiểm tra | Cách làm | Kết quả mong đợi |
|---|---|---|
| Ingest gọi được nguồn | `aws lambda invoke ... --payload '{"dry_run": true}'` (mục [5.4.1](../5.4-infrastructure/5.4.1-data-stack/)) | Mỗi mã khoảng 10 dòng, `error = null` |
| API không token | `curl "$API/forecasts/gold"` | `401` |
| API có token | `curl -H "Authorization: $ID_TOKEN" "$API/forecasts/gold"` | `200`, có `base_date`, `model_version` |
| Health | `curl "$API/health"` | `200` |
| CORS | Mở web trên Amplify, xem tab Network của trình duyệt | Preflight trả đúng origin, không có lỗi CORS |
| Chatbot | Hỏi một câu dự báo và một câu tin tức trên web | `200`; số trong câu trả lời trùng số trên trang; tin có link |
| Độ tin cậy | Gọi `/accuracy/gold` | `200`, `data_status = collecting` (khi chưa đủ 20 ngày cơ sở) hoặc `ready` |

#### Kiểm tra lượt chạy 19:15

```bash
# log của predict: lấy dòng REPORT (thời gian, bộ nhớ) và các mã sự kiện
aws logs tail /aws/lambda/cpaws-dev-predict --since 90m --format short --profile cpaws-dev \
  --filter-pattern "?REPORT ?PREDICT_DONE ?PREDICT_FAILED ?PREDICT_ALREADY_PUBLISHED ?PREDICT_SHADOW"

# trạng thái các alarm
aws cloudwatch describe-alarms --profile cpaws-dev \
  --query "MetricAlarms[].[AlarmName,StateValue]" --output text
```

Dòng `PREDICT_DONE` có giá phiên gần nhất, nên không dán nguyên dòng log này vào repo public hay PR.

#### Các ca lỗi đã kiểm

| Ca lỗi | Hành vi | Bằng chứng |
|---|---|---|
| Một nguồn giá lỗi | Tài sản khác vẫn cập nhật; ingest báo `IngestSourceFailed`, alarm gửi email | Test với nguồn lỗi 7 phiên liên tiếp, lần chạy sau tự tải bù |
| Chạy predict hai lần cùng ngày | Lần sau ghi log `PREDICT_ALREADY_PUBLISHED`, không ghi đè | Test `PutItem` có điều kiện |
| Quá 20 câu hỏi/ngày | `/chat` trả `429 rate_limited` | Test giao dịch DynamoDB có điều kiện |
| Hết thời gian trước khi gọi model | `/chat` trả `504` trước khi tính lượt | Test hạn thời gian |
| Ghi đè kết quả test2020 | Ghi lại cùng key bị từ chối (HTTP 412), xóa object bị chặn bởi explicit deny | Kiểm bằng AWS CLI ngày 04/11 và 11/11/2026 |

#### Giám sát

Stack `CpawsOps` tạo 6 CloudWatch alarm. Alarm nào cũng báo khi có **≥ 1 lỗi trong 5 phút** và gửi email qua SNS topic `cpaws-dev-alarms`:

| Alarm | Theo dõi |
|---|---|
| `cpaws-dev-ingest-errors` | Lỗi của Lambda ingest: nguồn chính lỗi hoặc Lambda crash |
| `cpaws-dev-predict-errors` | Lỗi của Lambda predict: có tài sản bị bỏ qua hoặc Lambda crash |
| `cpaws-dev-predict-shadow-errors` | Metric filter trên log predict: shadow S1a lỗi, champion vẫn phát hành |
| `cpaws-dev-accuracy-job-errors` | Lỗi của job độ tin cậy |
| `cpaws-dev-chat-api-errors` | Lambda chat timeout hoặc crash |
| `cpaws-dev-chat-api-internal-errors` | Metric filter: handler chat trả `500` vì lỗi không lường trước |

Mọi log group của Lambda giữ 1 tháng.

#### Độ tin cậy đo trên dữ liệu chạy thật

Job `cpaws-dev-accuracy-job` chạy lúc 20:45 giờ Việt Nam, thứ Ba–thứ Bảy. Job so các dự báo đã phát hành với giá thật đã biết, rồi tính `coverage_80`, `width_80` và các chỉ số khác trong 8 tuần gần nhất. Khi chưa đủ 20 ngày cơ sở, web hiện "Đang thu thập (n/20)" thay cho con số. Lượt theo lịch đầu tiên chạy ngày 12/11/2026: 4/4 tài sản hoàn tất trong 30,3 giây, alarm ở trạng thái OK, mức 9/20. Đến lượt ngày 21/11/2026, lượt cuối trước khi đóng băng, cả 4 tài sản đã ở mức 16/20.

#### Giới hạn còn lại

- Alarm không phát hiện trường hợp lịch **không chạy**, vì khi không có dữ liệu thì alarm coi là bình thường. Hiện nhóm kiểm tay sau mỗi lượt 19:15. Hướng sửa: thêm alarm trên số lần gọi Lambda ingest, báo khi 3 ngày liên tiếp không có lần chạy nào.
- Chỉ có một môi trường `dev`, deploy tay từ một máy, chưa có staging.

![CloudWatch alarms](/images/5-Workshop/5.6-alarms.png)

*6 alarm của CPAWS trong CloudWatch đều OK và đã bật Actions; mỗi alarm báo khi có từ 1 lỗi trở lên trong 5 phút.*

![Trang Độ chính xác](/images/5-Workshop/5.6-reliability.png)

*Phần đầu trang Độ chính xác: các tài sản đang ở trạng thái "Đang thu thập (16/20)", tính từ 29/10/2026, nên web chưa hiện kết quả chính thức.*
