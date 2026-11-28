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
4. **Tính lượt hỏi** bằng một lần ghi DynamoDB có điều kiện vào bảng `chat-usage`. Giới hạn là 20 lượt/người/ngày và 100 lượt/ngày cho cả hệ thống; hết lượt thì trả `429`. Bộ đếm làm mới lúc 00:00 giờ Việt Nam, và web hiện số lượt còn lại ngay dưới ô nhập. Bảng chỉ lưu bộ đếm, không lưu câu hỏi.
5. **Gọi Bedrock Converse** với system prompt `chatbot-haiku-v2`, hai tool, tối đa 3 vòng gọi tool. Từ bản `v1.0.1`, Lambda gửi thêm một khối `system` thứ hai ghi ngày giờ hiện tại theo giờ Việt Nam (UTC+7).
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
| Quyền IAM | `bedrock:InvokeModel` cho Claude Haiku 4.5 và Nova 2 Lite (Nova đã được cấp cho router nhưng chưa dùng). Theo hướng dẫn global cross-Region inference, mỗi model có 3 statement: inference profile, model trong region và model global; hai statement sau chỉ có hiệu lực khi đi qua đúng inference profile (điều kiện `bedrock:InferenceProfileArn`). `dynamodb:Query` trên bảng `forecasts` và `news-labels`; `dynamodb:GetItem`, `dynamodb:UpdateItem` trên bảng `chat-usage` để đếm lượt hỏi |
| Không dùng | Knowledge Bases, embedding, fine-tune. Tin tức đọc trực tiếp từ dữ liệu đã gắn nhãn |

Dùng inference profile global thì Bedrock có thể xử lý request ở region khác. Nhóm ghi rõ trong ADR-0004 rằng không cam kết mọi request được xử lý hoàn toàn trong Singapore.

#### Chatbot biết ngày hiện tại (v1.0.1)

Đến bản `v1.0.0`, model không biết hôm nay là ngày nào. Khi nhóm hỏi thử ngày 23/11/2026, cụm "từ 9/11 đến 20/11 năm nay" bị hiểu thành năm 2024, còn "hôm nay" bị nhầm với ngày cơ sở của dự báo. Bản `v1.0.1` sửa lỗi này bằng một khối `system` thứ hai, tạo lúc nhận request, ghi thứ, ngày và giờ theo UTC+7. Nội dung prompt `chatbot-haiku-v2` không đổi vì đã khóa sau benchmark. Thay đổi đến sau benchmark nên chỉ được kiểm bằng unit test và hỏi thử trên Production ngày 24/11/2026: chatbot trả lời đúng ngày, năm hiện tại và tìm đúng tin của năm 2026.

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

#### Chatbot trên web

Ảnh chụp trên Production ngày 24/11/2026 (bản v1.0.1). Thông tin tài khoản, mã nội bộ của từng câu trả lời và các mức giá USD phía sau khung chat đã được che.

![Hỏi dự báo](/images/5-Workshop/5.5-chat-forecast-news1.png)

*Câu hỏi về dự báo vàng 7 phiên tới: chatbot nêu khoảng 80%, trung vị và xác suất tăng theo %, ghi rõ chuỗi đại diện GLD và mốc thống kê 252 phiên, kèm câu disclaimer.*

![Hỏi tin tức](/images/5-Workshop/5.5-chat-forecast-news2.png)

*Câu hỏi về tin Fed: chatbot trả lời không tìm thấy tin Fed nào trong 7 ngày (17/11–24/11/2026). Thực ra bộ tin có biên bản FOMC đăng ngày 18/11 với mức tác động 2, nên đây là một lần bỏ sót; lỗi này được ghi ở mục "Hạn chế còn lại sau khi đóng băng" bên dưới.*

![Từ chối tư vấn](/images/5-Workshop/5.5-chat-refuse-advice.png)

*Câu hỏi "có nên mua vàng không?": chatbot từ chối tư vấn đầu tư và chỉ nhắc lại số dự báo. Dưới ô nhập là số lượt hỏi còn lại trong ngày và giờ làm mới lượt.*

#### Hạn chế còn lại sau khi đóng băng

Ngày 23–24/11/2026, nhóm hỏi thử chatbot trên Production với bản v1.0.1 và ghi nhận:

- **Chưa trả lời được câu hỏi về biến động giá đã qua**, ví dụ vì sao giá giảm trong một khoảng ngày. Chatbot chưa có tool đọc lịch sử giá nên chuyển sang nói về dự báo các phiên tới, rồi nhắc một tin cùng thời điểm. Cách trả lời này dễ khiến người đọc hiểu nhầm là tin đó gây ra mức giảm.
- **Gọi tên xu hướng khi xác suất tăng chỉ quanh 50%.** Chatbot vẫn nói "xu hướng giảm", trong khi ở mức này nên nói là chưa rõ xu hướng. Web chỉ hiện nhãn xu hướng khi xác suất tăng từ 60% trở lên hoặc từ 40% trở xuống.
- **Hiểu sai hai đầu của khoảng 80% (q10/q90).** Chatbot coi chúng là mức sàn và mức trần, viết thành "giảm tối đa" hay "tăng tối đa". Thực ra giá vẫn có thể ra ngoài khoảng này.
- **Tự thêm nhận định không có trong tin**, chẳng hạn "lãi suất cao làm vàng kém hấp dẫn vì chi phí cơ hội tăng".
- **Ngày đăng tin lệch một ngày.** Câu trả lời ghi ngày theo giờ UTC, còn thẻ Nguồn trên web đổi sang giờ Việt Nam.
- **Bỏ sót tin đã có trong bộ tin.** Với câu hỏi không nêu tài sản, như "Có tin gì mới của Fed không?", chatbot có lúc trả lời không tìm thấy, dù biên bản FOMC ngày 18/11 nằm trong khoảng 7 ngày và có mức tác động 2. Khi hỏi rõ tài sản và khoảng ngày, chatbot trả đúng tin này.

Các lỗi nội dung trên đến từ prompt v2 đang chạy trên Lambda. Sau khi đóng băng, nhóm không sửa prompt hay code nữa mà chỉ ghi lại. Hướng cải tiến về sau:

- Thêm tool đọc lịch sử giá.
- Quy định trong prompt cách gọi xu hướng và cách diễn giải khoảng dự báo.
- Thống nhất múi giờ khi hiển thị ngày đăng tin.
