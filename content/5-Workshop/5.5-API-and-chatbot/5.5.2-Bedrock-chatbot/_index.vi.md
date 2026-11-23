---
title: "Chatbot Bedrock"
date: 2025-09-09
weight: 2
chapter: false
pre: " <b> 5.5.2 </b> "
---

Chatbot giúp người dùng đọc dự báo CPAWS bằng tiếng Việt: dự báo của một tài sản ở một kỳ hạn nói gì, các khái niệm như "khoảng 80%" hay "xác suất tăng" nghĩa là gì, và gần đây có tin Fed/EIA nào. Đây là phần mình phụ trách trong CPAWS.

Chatbot **không tự dự báo**. Mọi con số đều lấy từ bản ghi dự báo đã phát hành thông qua tool, nên câu trả lời và trang dự báo luôn cùng một số. Chatbot cũng không bao giờ tư vấn đầu tư.

#### Một request được xử lý thế nào

1. Đầu tiên API Gateway xác thực ID token của Cognito. Từ đó về sau, hệ thống chỉ biết người dùng qua giá trị `sub` trong token; email không được dùng.
2. Lambda `cpaws-dev-chat-api` (Python 3.12, 512 MB, 28 giây) kiểm tra body. `question` phải dài từ 1 đến 500 ký tự; `context.asset` (trang người dùng đang mở) có thể bỏ trống; `history` chứa được nhiều nhất 6 lượt. Hội thoại nằm ở trình duyệt, server không giữ. Sai dạng thì trả `400` và không tính lượt.
3. **Hạn chót.** API Gateway bỏ request sau 29 giây tính từ `requestTimeEpoch`, nên Lambda cũng đếm từ đúng mốc đó và chừa lại 4 giây. Như vậy thời gian của authorizer và cold start đã nằm sẵn trong quỹ thời gian. Khi thời gian còn lại quá ngắn cho thêm một lần gọi model, Lambda tự trả `504` thay vì để gateway cắt kết nối.
4. **Giới hạn lượt.** Mỗi người được hỏi 20 câu một ngày, cả hệ thống 100 câu, bộ đếm bắt đầu lại lúc nửa đêm giờ Việt Nam. Hai bộ đếm trong `cpaws-dev-chat-usage` được tăng trong cùng một `TransactWriteItems`, mỗi bộ có điều kiện riêng; chỉ cần một điều kiện không thỏa là cả giao dịch bị hủy, không bộ đếm nào nhích, và người dùng nhận `429 rate_limited`.
5. **Gọi model.** Bedrock Converse với Claude Haiku 4.5, system prompt `chatbot-haiku-v2`, `temperature` 0, `max_tokens` 1500, tối đa ba vòng gọi tool.
6. **Trả kết quả.** `answer` (văn bản thuần), `forecasts_used`, `citations`, `quota`, `disclaimer` và vài cờ. Một dòng log JSON ghi tool đã gọi, số token và thời gian, nhưng không ghi nội dung câu hỏi.

Câu trả lời trả một lần, không streaming. Cách này giữ nguyên được Cognito authorizer và Lambda Python, còn thời gian trả lời đo được nằm xa dưới mốc 29 giây.

#### Hai tool

| Tool | Trả về gì | Vì sao thiết kế như vậy |
|---|---|---|
| `get_forecast(asset, horizon?)` | Bản ghi mới nhất trong `cpaws-dev-forecasts`, định dạng bằng chính hàm của API dự báo rồi rút gọn: chỉ q10, q50, q90 theo %, xác suất tăng theo %, độ rộng khoảng 80%; giá dầu theo USD/thùng; **không trả mức giá vàng, bạc** | Ở lần benchmark đầu, model nhầm cặp phân vị, tự nhân phần trăm và nêu giá vàng. Đưa cho model ít số hơn, tính sẵn hơn thì các lỗi đó hết |
| `get_news(asset, from_date?, to_date?, min_impact?)` | Tin Fed/EIA đã gắn nhãn: tiêu đề, tóm tắt, link, nguồn, ngày, mức tác động 0–3, chiều tác động; mặc định 7 ngày, `min_impact` 2, tối đa 10 tin | Chatbot phải trích link cho mọi tin, nên tin không có link hợp lệ bị bỏ |

Nhãn tin làm bằng tay: 2–3 thông cáo Fed/EIA mỗi tuần, đóng gói dạng JSON cùng Lambda (ADR-0013). Nhóm đã thử gắn nhãn tự động bằng Nova 2 Lite nhưng không qua được bước kiểm (đúng chiều 52,4% so với mục tiêu 80%), nên chọn gắn nhãn tay đến hết dự án. Đổi bộ tin thì phải deploy lại `CpawsApi`. Bộ tin mới nhất lên Lambda cùng bản v1.0.0 ngày 22/11: biên bản FOMC họp 27–28/10 (cho vàng, bạc) và báo cáo dầu hằng tuần của EIA ngày 18/11 (cho Brent, WTI).

#### Quyền truy cập Bedrock và IAM

Model được gọi qua global cross-Region inference profile `global.anthropic.claude-haiku-4-5-20251001-v1:0` từ `ap-southeast-1` (xem [5.2](../../5.2-prerequiste/)). Vì profile là global, nhóm không cam kết mọi request đều được xử lý trong Singapore.

Role của Lambda có đúng những quyền cần:

+ `bedrock:InvokeModel` trên ba ARN của profile và model, không có quyền streaming;
+ `dynamodb:Query` trên bảng dự báo và bảng tin;
+ `dynamodb:UpdateItem` và `GetItem` trên `cpaws-dev-chat-usage`.

#### Các lớp bảo vệ không phụ thuộc model

Model không phải lúc nào cũng nghe theo prompt, nên những quy tắc quan trọng nhất được kiểm thêm một lần nữa bằng code:

+ Một tin chỉ vào được `citations` khi `get_news` đã trả nó trong request này và link của nó có mặt trong câu trả lời. Link bấm được trên web chỉ lấy từ `citations`; chữ trong câu trả lời không bao giờ được biến thành link.
+ Câu trả lời nào có URL không thuộc tin của request đó đều bị ghi log `citation_link_mismatch` (request ID, phiên bản prompt và các URL, không ghi câu hỏi hay câu trả lời).
+ `disclaimer` và `forecasts_used` luôn có trong response, để web hiện câu disclaimer và nhãn "dữ liệu mẫu" kể cả khi câu trả lời quên.
+ `history` từ client không bao giờ là nguồn số liệu; muốn nêu số, model phải gọi lại `get_forecast`.
+ Không có gì nội bộ lọt ra response, dù request thành công hay lỗi: không Account ID, ARN, tên bảng, model ID, system prompt hay stack trace.

Bedrock Guardrails được thiết kế là lớp thứ ba, nhưng guardrail vẫn ở trạng thái `DRAFT` và chưa gắn vào Lambda.

#### Deploy và chạy thử

Bản đang chạy trên `/chat` là **v1.0.0** (tag `v1.0.0`, commit `ba3d9f4`), deploy cùng `CpawsApi` ngày 22/11/2026, vẫn dùng prompt `chatbot-haiku-v2`. Bản vá tiếp theo, v1.0.1, gửi thêm một khối `system` thứ hai nêu ngày giờ hiện tại theo giờ Việt Nam để model không còn nhầm "hôm nay" với ngày cơ sở của dự báo; bản thân prompt v2 đã khóa không đổi. Bản này đã được approve nhưng chưa merge và chưa deploy.

Code chat, hai file prompt v2 và các file tin JSON được chép vào asset của Lambda lúc `cdk synth`. Mọi thay đổi ở các file này đi theo lần deploy `CpawsApi`:

```bash
cdk diff CpawsApi -c allowed_origin=ORIGIN_WEB --profile cpaws-dev
cdk deploy CpawsApi -c allowed_origin=ORIGIN_WEB --profile cpaws-dev
```

Chạy thử với ID token (dán bằng `read -rs ID_TOKEN && export ID_TOKEN`):

```bash
curl -sS -X POST "$API_URL/chat" \
  -H "Authorization: $ID_TOKEN" -H "Content-Type: application/json" \
  -d '{"question": "Dự báo bạc 5 phiên tới thế nào?", "context": {"asset": "silver", "page": "product"}}'
```

| Mã | `error` | Ý nghĩa |
|---|---|---|
| 400 | `invalid_body`, `empty_question`, `question_too_long`, `invalid_context`, `invalid_history` | Request sai, không tính lượt |
| 429 | `rate_limited` | Hết lượt trong ngày (`reset_at`, `Retry-After`) |
| 503 | `model_unavailable` | Bedrock vẫn lỗi sau một lần thử lại |
| 504 | `timeout` | Không còn đủ thời gian cho lần gọi model tiếp theo |
| 500 | `internal_error` | Lỗi khác; chi tiết chỉ nằm trong CloudWatch kèm `request_id` |

Muốn tìm câu trả lời có link lạ, chạy truy vấn này trong CloudWatch Logs Insights trên log group của chat:

```
fields @timestamp, request_id, prompt_version
| filter event = "citation_link_mismatch"
| sort @timestamp desc
```

<!-- ẢNH: khung chat trên trang bạc, câu trả lời có khoảng 80%, xác suất tăng và disclaimer -->
[ẢNH: khung chat trên trang bạc kèm disclaimer]

#### Chatbot được đánh giá thế nào

Mỗi lần đổi prompt đều được đo bằng benchmark gọi model Bedrock thật, với tool đọc một fixture cố định. Câu hỏi, prompt, fixture và code tool được khóa bằng SHA-256, ngưỡng đặt trước khi chạy, và mỗi bộ chính thức chỉ chạy đúng một lần. Mình chấm toàn bộ câu trả lời (Claude gợi ý điểm kèm lý do), còn Khải chấm độc lập một mẫu, không dùng model.

| Lần chạy | Kết quả | Ngưỡng | Quyết định |
|---|---:|---:|---|
| Prompt v2, bộ B (30 câu cần tool), 30/10 | 14/30 | ≥ 25 | Chưa đạt; vẫn giữ trên `/chat` vì chưa prompt nào sau đó đạt điều kiện |
| Router theo luật, bộ giữ riêng, 04/11 | 26/30 | ≥ 27 (90%) | Không dùng; mọi câu đi Haiku |
| Prompt v4, bộ B mới, 11/11 | 17/30 | ≥ 25 | Không đạt; giữ v2 |
| Prompt v4, bộ A mới (8 câu đơn giản), 11/11 | 5/8 | ≥ 7 | Không đạt |

Qua mọi lần chạy chính thức: **0 vi phạm chính sách**, và model chưa lần nào làm theo chỉ dẫn chèn trong nội dung tin. Lỗi còn lại chủ yếu là đọc số (bỏ dấu âm, so sánh hai số âm ngược chiều, gọi q90 là "tăng tối đa") và thiếu phần bắt buộc. Trên bộ B với v2, thời gian trả lời trung vị là 4.023 ms, p90 là 5.733 ms.

**Chi phí.** Khoảng 0,012 USD mỗi câu với Haiku. Với trần 100 câu/ngày, chatbot tiêu tối đa khoảng 1,2 USD/ngày, tức khoảng 36 USD/tháng trong trường hợp xấu nhất, dưới ngân sách 50 USD/tháng của dự án.
