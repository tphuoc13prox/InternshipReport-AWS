---
title: "Chatbot với Amazon Bedrock"
date: 2026-11-20
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---
#### Mục tiêu và ràng buộc

Chatbot giúp người dùng đọc dự báo bằng tiếng Việt: hỏi về vàng, bạc, Brent, WTI ở kỳ hạn 1–7 phiên, hỏi khái niệm (khoảng 80%, xác suất tăng) và hỏi tin tức liên quan. Chatbot **không tự dự báo**: mọi con số lấy từ bản ghi đã phát hành, nên trùng với số trên trang dự báo. Chatbot có ba ràng buộc:

1. **Không tư vấn đầu tư.** Không khuyên mua, bán, giữ hay phân bổ vốn.
2. **Số liệu phải có nguồn.** Không dùng số ngoài kết quả tool, không tự tính chênh lệch hay quy đổi.
3. **Vừa với API Gateway:** API Gateway cắt request ở 29 giây, nên câu trả lời phải xong trước mốc đó.

#### Luồng `POST /chat`

1. API Gateway kiểm tra ID token bằng Cognito authorizer. Người dùng được xác định bằng claim `sub`, không dùng email.
2. Lambda `cpaws-dev-chat-api` (Python 3.12, 512 MB, timeout 28 giây) kiểm tra request: câu hỏi dài 1–500 ký tự, lịch sử hội thoại tối đa 6 lượt do client gửi lên. Server không lưu hội thoại.
3. **Kiểm tra hạn thời gian.** Lambda tính hạn chót từ thời điểm API Gateway nhận request (29 giây trừ 4 giây dự phòng). Nếu không còn đủ thời gian cho một lần gọi model thì trả `504` trước khi tính lượt.
4. **Tính lượt hỏi** bằng một lần ghi DynamoDB có điều kiện vào bảng `chat-usage`. Giới hạn là 20 lượt/người/ngày và 100 lượt/ngày cho cả hệ thống; hết lượt thì trả `429`. Bảng chỉ lưu bộ đếm, không lưu câu hỏi.
5. **Gọi Bedrock Converse** với system prompt `chatbot-haiku-v2`, hai tool, tối đa 3 vòng gọi tool.
6. **Trả kết quả:**
   - câu trả lời dạng văn bản thuần;
   - `citations` (danh sách tin đã trích);
   - `forecasts_used` (tài sản, ngày cơ sở, phiên bản mô hình, cờ dữ liệu mẫu);
   - số lượt hỏi còn lại và câu disclaimer.
7. **Ghi một dòng log JSON:** nhánh, tool đã gọi, số token, thời gian. Không ghi nguyên văn câu hỏi.

#### Model và cách gọi

| Mục | Cấu hình |
|---|---|
| Model | Claude Haiku 4.5 qua inference profile **global** (cross-Region inference), gọi từ `ap-southeast-1` |
| API | Converse (không stream), `temperature` 0, `maxTokens` 1500, `toolChoice: auto` |
| Quyền IAM | Chỉ `bedrock:InvokeModel` trên đúng ARN của inference profile và model; `dynamodb:Query` trên bảng `forecasts` |
| Không dùng | Knowledge Bases, embedding, fine-tune. Tin tức đọc trực tiếp từ dữ liệu đã gắn nhãn |

Dùng inference profile global thì Bedrock có thể xử lý request ở region khác. Nhóm ghi rõ trong ADR-0004 rằng không cam kết mọi request được xử lý hoàn toàn trong Singapore.

#### Router: một kết quả âm có kiểm soát

Thiết kế ban đầu (ADR-0004) chia câu hỏi làm hai nhánh. Câu đơn giản (chào hỏi, khái niệm) đi **Amazon Nova 2 Lite** để tiết kiệm. Câu cần số liệu đi **Claude Haiku 4.5** có tool. Router chọn nhánh theo luật, không gọi LLM. Router được thử như sau:

- 9 luật được **khóa trước** khi đo.
- Bộ 30 câu giữ riêng do người khác soạn; người viết luật không được xem trước.
- Ngưỡng chốt trước: độ chính xác ≥ 90%, tối đa 1 câu cần Haiku bị đưa nhầm sang Nova.
- Chạy đúng một lần.

Kết quả: độ chính xác **26/30 (86,7%), không đạt**. Không câu nào cần Haiku bị đưa sang Nova; 4 câu sai đều là câu đơn giản bị đưa sang Haiku, chỉ tốn thêm chi phí. Nhóm quyết định **không đưa router vào `/chat`: mọi câu đều đi Haiku**, và không sửa luật rồi chấm lại trên bộ đã xem. Chi phí khoảng 0,012 USD/câu. Với giới hạn 100 lượt/ngày, chi phí chatbot tối đa khoảng 1,2 USD/ngày.

#### Tool v2

| Tool | Nguồn | Trả về |
|---|---|---|
| `get_forecast(asset, horizon?)` | DynamoDB `cpaws-dev-forecasts`, `Query` với `Limit=1`. Định dạng bằng cùng hàm với Lambda `forecasts-api` | q10, q50, q90 theo %, xác suất tăng, độ rộng khoảng 80%, ngày cơ sở, `model_version`, cờ dữ liệu mẫu. Với vàng/bạc không trả mức giá |
| `get_news(asset, from_date?, to_date?, min_impact?)` | Thông cáo Fed và EIA đã gắn nhãn tay | Tối đa 10 tin, mỗi tin có tiêu đề, tóm tắt do nhóm viết, link, nguồn, ngày, mức tác động 0–3, hướng tác động |

Mỗi quy tắc rút gọn của tool v2 sửa một lỗi gặp ở lần benchmark trước. Ví dụ: trả sẵn xác suất tăng theo % vì model từng nhân 100 sai; chỉ trả q10/q90 vì model từng gọi nhầm cặp q05/q95 là "khoảng 80%".

#### Gọi Converse với tool use

```python
import boto3

bedrock = boto3.client("bedrock-runtime", region_name="ap-southeast-1")

TOOLS = [{"toolSpec": {
    "name": "get_forecast",
    "description": "Dự báo mới nhất (q10, q50, q90, xác suất tăng) của một tài sản",
    "inputSchema": {"json": {
        "type": "object",
        "properties": {
            "asset": {"type": "string", "enum": ["gold", "silver", "brent", "wti"]},
            "horizon": {"type": "integer", "minimum": 1, "maximum": 7},
        },
        "required": ["asset"],
    }},
}}]  # get_news khai báo tương tự

def ask(question: str, model_id: str, system_prompt: str, run_tool) -> str:
    messages = [{"role": "user", "content": [{"text": question}]}]
    for _ in range(3):  # tối đa 3 vòng gọi tool
        resp = bedrock.converse(
            modelId=model_id, system=[{"text": system_prompt}], messages=messages,
            toolConfig={"tools": TOOLS},
            inferenceConfig={"maxTokens": 1500, "temperature": 0},
        )
        msg = resp["output"]["message"]
        messages.append(msg)
        if resp["stopReason"] != "tool_use":
            return "".join(b.get("text", "") for b in msg["content"])
        # chạy từng tool model yêu cầu, gửi kết quả lại bằng toolResult
        results = [{"toolResult": {"toolUseId": b["toolUse"]["toolUseId"],
                                   "content": [{"json": run_tool(b["toolUse"]["name"], b["toolUse"]["input"])}]}}
                   for b in msg["content"] if "toolUse" in b]
        messages.append({"role": "user", "content": results})
    return "Mình chưa trả lời được câu này, bạn thử hỏi ngắn gọn hơn nhé."
```

#### Lớp bảo vệ phía server

Model có thể bỏ qua prompt, nên một số quy tắc được kiểm tra lại bằng code:

- `citations` chỉ gồm tin do `get_news` trả về **trong lượt này** và có link xuất hiện trong câu trả lời. Web chỉ tạo link từ `citations`, không tự biến chữ trong câu trả lời thành link.
- Câu trả lời có link không thuộc tin của tool thì bị ghi log `citation_link_mismatch` để tra lại.
- Câu trả lời hiển thị dạng văn bản thuần: không render Markdown hay HTML.
- Disclaimer và `forecasts_used` luôn có trong response, kể cả khi model quên.
- Response và thông báo lỗi không chứa Account ID, ARN, tên bảng, model ID hay system prompt.
- Nội dung tin là **dữ liệu không tin cậy**: prompt yêu cầu bỏ qua mọi chỉ dẫn nằm trong tin.

Nhóm **không bật Bedrock Guardrails** trong phạm vi đồ án (quyết định ngày 11/11/2026). Chủ đề tư vấn đầu tư và prompt attack được kiểm soát bằng quy tắc prompt, các lớp kiểm tra trên và bộ benchmark có câu kiểm tra chính sách. Guardrails là hướng cải tiến sau đồ án.

#### Đánh giá

- Benchmark gọi Bedrock thật, với tool chạy trên dữ liệu cố định.
- Trước khi chạy, câu hỏi, prompt, dữ liệu và code tool được **khóa bằng SHA-256**.
- Lượt chính thức chỉ chạy **một lần**; không sửa prompt rồi chấm lại trên cùng bộ câu.
- Prompt v4 không đạt lần chạy chính thức ngày 11/11/2026. Vì vậy Lambda giữ prompt v2. Chi phí cả lượt benchmark khoảng 0,73 USD.

<!-- ẢNH: khung chat trên web: một câu hỏi dự báo + một câu hỏi tin tức có trích nguồn (CHE con số giá) -->
<!-- ẢNH: câu hỏi xin tư vấn mua/bán và câu trả lời từ chối -->
