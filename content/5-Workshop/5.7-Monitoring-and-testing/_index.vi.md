---
title: "Giám sát, kiểm thử và rollback"
date: 2025-09-09
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Bước 1 – Alarm (`CpawsOps`)

`CpawsOps` tạo SNS topic `cpaws-dev-alarms` và sáu alarm CloudWatch. Cả sáu dùng chung một quy tắc: **từ 1 lần trở lên trong 5 phút**, chu kỳ không có dữ liệu coi là bình thường.

| Alarm | Metric | Báo khi |
|---|---|---|
| `cpaws-dev-ingest-errors` | `AWS/Lambda Errors` của ingest | Một nguồn giá chính lỗi hoặc Lambda crash |
| `cpaws-dev-predict-errors` | `AWS/Lambda Errors` của predict | Predict phải bỏ qua ít nhất một tài sản, hoặc bị crash |
| `cpaws-dev-predict-shadow-errors` | Metric filter trên log predict | S1a lỗi hoặc cấu hình sai. Vì champion vẫn được phát hành, metric `Errors` thông thường vẫn bằng 0 trong trường hợp này |
| `cpaws-dev-accuracy-job-errors` | `AWS/Lambda Errors` của job accuracy | Có tài sản `FAILED`, timeout hoặc crash. "Chưa đủ dữ liệu" không tính là lỗi |
| `cpaws-dev-chat-api-errors` | `AWS/Lambda Errors` của chat | Timeout hoặc crash |
| `cpaws-dev-chat-api-internal-errors` | Metric filter `CPAWS/ChatInternalErrors` trên log chat | Handler bắt được lỗi không lường trước và trả `500`. Lỗi 4xx, 429 và lỗi Bedrock có mã riêng là hành vi đã thiết kế, không báo |

Hai metric filter cần có vì các lỗi đó được bắt ngay trong code, nên Lambda không bao giờ báo chúng là `Errors`.

Email nhận cảnh báo truyền lúc deploy, không ghi vào repo:

```bash
cdk diff CpawsOps -c alarm_email=EMAIL_NHAN_CANH_BAO --profile cpaws-dev
cdk deploy CpawsOps -c alarm_email=EMAIL_NHAN_CANH_BAO --profile cpaws-dev
```

Sau lần deploy đầu, người nhận phải bấm **Confirm subscription** trong email của SNS. Các lần deploy sau cũng phải truyền `alarm_email`; thiếu thì subscription bị gỡ.

```bash
aws cloudwatch describe-alarms --profile cpaws-dev \
  --query "MetricAlarms[].[AlarmName,StateValue]" --output text
```

<!-- ẢNH: CloudWatch console, danh sách Alarms có sáu alarm cpaws-dev-* ở trạng thái OK -->
[ẢNH: CloudWatch – sáu alarm `cpaws-dev-*` ở trạng thái OK]

#### Bước 2 – Kiểm tay sau mỗi lượt chạy

Alarm chỉ báo khi có lỗi. Nếu lịch không kích hoạt thì không có dữ liệu, mà không có dữ liệu thì coi là bình thường, nên không ai nhận được email. Vì vậy nhóm kiểm tay sau mỗi lượt 19:15:

+ log predict: `PREDICT_DONE` với `written` hoặc `already_published`, và `Duration`;
+ số file shadow trong `eval/shadow/s1a-l094/dt=<ngày>/` (mong đợi 4);
+ trạng thái của mọi alarm;
+ sau 20:45, log job accuracy và `State` của schedule.

Lệnh cụ thể ở [5.4.2](../5.4-data-pipeline/5.4.2-ingest-and-predict/) và [5.4.3](../5.4-data-pipeline/5.4.3-rolling-accuracy/). Hướng sửa đã lên kế hoạch là một alarm trên `Invocations` của ingest, chu kỳ một ngày, coi thiếu dữ liệu là vi phạm và chỉ báo sau ba ngày liền không chạy, vì Chủ nhật và thứ Hai (giờ UTC) vốn không có lịch.

#### Bước 3 – Kiểm thử trước khi deploy

| Lớp | Chạy gì | Ở đâu |
|---|---|---|
| Python | `ruff check`, `ruff format --check`, `pytest -q` (job `test-and-lint`) | GitHub Actions ở mọi push và pull request |
| Hạ tầng | `tests/test_infra_*.py` synth app CDK và kiểm quyền IAM, biến môi trường, lịch chạy, alarm | Cùng job. Job cài `aws-cdk-lib` và chạy bước kiểm `import aws_cdk` trước, để test CDK không thể bị bỏ qua mà CI vẫn báo xanh |
| Web | `npm ci`, `typecheck`, `test`, `build` | Cùng workflow |
| Pre-commit | gitleaks và `detect-aws-credentials` (vì repo để public), cùng ruff, ruff-format, `check-added-large-files` và `end-of-file-fixer` | Mỗi lần commit |
| Chatbot | Benchmark trên Bedrock thật với câu hỏi và ngưỡng đã khóa | Chạy tay, xem [5.5.2](../5.5-api-and-chatbot/5.5.2-bedrock-chatbot/) |

CI không bao giờ deploy và GitHub không giữ quyền AWS nào. Nhánh `developer` cần ít nhất một approve để merge; thay đổi IAM và hạ tầng được người phụ trách bảo mật của nhóm review ngay trong pull request.

Hai ràng buộc được kiểm ngay khi synth app CDK, để lỗi bị chặn trước khi lên AWS:

+ một test kiểm hằng số lịch trong `infra/stacks/shared.py` khớp quy tắc thời điểm có dữ liệu trong `src/data/config.py`, cũng là quy tắc backtest dùng;
+ CDK từ chối synth nếu `MODEL_VERSION` lệch với bất kỳ giá trị nào trong `MODEL_VERSIONS`, nên image predict cũ vẫn chạy đúng champion. Cũng vì vậy mà đổi champion phải sửa `app.py` qua pull request.

#### Bước 4 – Rollback

Mọi thay đổi có rủi ro đều quay lại được bằng một lần deploy hoặc một lần upload, không cần build lại image:

| Tình huống | Lệnh | Build lại? |
|---|---|---|
| Shadow S1a lỗi hoặc làm predict chậm | `cdk deploy CpawsData -c shadow_models=off --profile cpaws-dev` | Không |
| Image predict mới lỗi | `cdk deploy CpawsData -c predict_image_tag=ce77bcb -c shadow_models=off --profile cpaws-dev` | Không, image cũ còn trong ECR |
| Image ingest mới lỗi | `cdk deploy CpawsData -c ingest_image_tag=TAG_CU --profile cpaws-dev` | Không |
| Job accuracy chạy sai | `cdk deploy CpawsData -c accuracy_schedule=off --profile cpaws-dev` | Không |
| Web trên Amplify không gọi được API | `cdk deploy CpawsApi -c allowed_origin=http://localhost:5173 --profile cpaws-dev` | Không |
| Bản web mới lỗi, hoặc phải ẩn giá kim loại | Upload lại zip bản trước, hoặc build với `VITE_SHOW_METAL_PRICES=false` | Chỉ build web |

{{% notice warning %}}
ECR chỉ giữ 5 image mới nhất mỗi repo. Trước khi push image mới, kiểm tag rollback còn trong repo, hoặc chọn tag rollback khác.
{{% /notice %}}
