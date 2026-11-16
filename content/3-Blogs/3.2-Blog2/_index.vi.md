---
title: "Blog 2"
date: 2026-11-16
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Tool use với Bedrock Converse API: Model gọi đúng tool rồi, sao vẫn trả lời sai?

bởi Phan Tấn Phước | ngày 16/11/2026 | đăng tại [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Bài gốc trên Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2339395856825373/)

*Bài tự viết, rút ra từ kinh nghiệm làm chatbot trên Amazon Bedrock; ví dụ trong bài (chatbot thời tiết) là ví dụ minh họa. Nội dung dưới đây giữ đúng câu chữ của bài đã đăng, chỉ trình bày lại bằng markdown.*

Thời gian gần đây mình làm chatbot trên Amazon Bedrock, dùng Converse API để cho model gọi tool lấy dữ liệu thật. Có một tình huống làm mình bất ngờ: model gọi đúng tool, tool trả về đúng dữ liệu, nhưng câu trả lời cuối vẫn sai. Bài này mình ghi lại cách vòng lặp tool use hoạt động và những gì rút ra khi thiết kế tool. Để dễ hình dung, mình dùng ví dụ một chatbot thời tiết; sơ đồ đánh số ở Hình 1.

### 1. Model không tự gọi tool

Điều đầu tiên cần nắm: trên Bedrock, model không trực tiếp chạy tool. Model chỉ nói "tôi cần gọi tool này với input này", còn code của bạn chạy tool rồi gửi kết quả lại. Một lượt hỏi đi qua 5 bước:

- **(1)** Ứng dụng gọi Converse, gửi messages kèm `toolConfig`. Mỗi tool có `toolSpec` gồm `name`, `description` và `inputSchema` (JSON Schema).
- **(2)** Model trả về `stopReason = "tool_use"`. Trong message có khối `toolUse` gồm `toolUseId`, `name` và `input`, ví dụ `get_weather_forecast` với `{"city": "Ho Chi Minh", "days": 3}`.
- **(3)** Code của bạn chạy hàm tương ứng, ví dụ Lambda đọc dữ liệu từ DynamoDB.
- **(4)** Gửi lại một message có role `"user"`, chứa khối `toolResult` với đúng `toolUseId` đó, `content` (json hoặc text) và `status`.
- **(5)** Gọi Converse lần nữa. Model có thể đòi thêm tool (quay lại bước 2) hoặc trả lời xong với `stopReason = "end_turn"`.

![Sơ đồ vòng lặp tool use với Bedrock Converse API](/images/3-Blog/blog2_figure1.png)

*Hình 1. Sơ đồ vòng lặp tool use với ví dụ chatbot thời tiết (5 bước được đánh số), mình tự vẽ dựa trên tài liệu AWS*

`toolUseId` là thứ để model ghép kết quả với đúng yêu cầu. Khi người dùng hỏi kiểu "mai trời Sài Gòn với Hà Nội thế nào?", model có thể đòi nhiều tool trong cùng một lượt; khi đó nên chạy hết rồi gom tất cả `toolResult` vào một message.

### 2. Vòng lặp phải có "phanh"

Ví dụ trong tài liệu thường chỉ gọi tool một lần. Khi đưa vào chạy thật, nên thêm 3 thứ:

- **Giới hạn số vòng,** ví dụ tối đa 3 vòng tool. Quá giới hạn mà model vẫn đòi tool thì trả một câu cố định ("Mình chưa trả lời được câu này, bạn thử hỏi cụ thể hơn nhé"), không trả câu dạo đầu dở dang kiểu "Để mình tra...".
- **Hạn chót cho cả lượt hỏi.** Nếu chatbot đứng sau API Gateway (REST API), integration timeout mặc định tối đa là 29 giây (Regional API có thể xin tăng). Trước mỗi lần gọi model, kiểm tra thời gian còn lại có đủ cho một lần gọi (timeout kết nối + timeout đọc của client) không. Không đủ thì Lambda chủ động trả lỗi timeout đúng định dạng, thay vì để API Gateway cắt ngang.
- **Xử lý mọi `stopReason`,** không chỉ `"tool_use"` và `"end_turn"`. Converse còn có `max_tokens`, `stop_sequence`, `guardrail_intervened`, `content_filtered`, `malformed_model_output`, `malformed_tool_use`, `model_context_window_exceeded`. Ví dụ `max_tokens` thì đánh dấu câu trả lời bị cắt, `guardrail_intervened` thì trả câu từ chối cố định.

Mã giả cho mỗi lượt hỏi:

```python
messages = history + [câu hỏi]
for vòng in 0..MAX_ROUNDS:
    if thời_gian_còn_lại() < thời_gian_một_lần_gọi: trả lỗi timeout
    resp = Converse(modelId, system, messages, toolConfig)
    messages.append(resp.output.message)
    if resp.stopReason != "tool_use": break
    if vòng == MAX_ROUNDS: trả câu cố định; dừng
    results = []
    for tu in các khối toolUse:
        out = chạy_tool(tu.name, tu.input)
        results.append(toolResult(tu.toolUseId, json=out, status="error" if lỗi else "success"))
    messages.append({role: "user", content: results})
```

Một điều nên biết về chi phí: Converse không lưu hội thoại giúp bạn, nên mỗi vòng tool là một lần gọi model và phải gửi lại toàn bộ messages, gồm cả kết quả tool trước đó. Token đầu vào vì vậy tăng nhanh theo số vòng, thường lớn hơn token đầu ra nhiều lần. Giới hạn số vòng cũng là giới hạn chi phí.

### 3. Tool lỗi thì báo lỗi, đừng để model đoán

- Tool trả lỗi có cấu trúc, ví dụ `{"error": "not_found", "message": "Chưa có dữ liệu dự báo cho thành phố này"}`, và đặt `status = "error"` trong `toolResult`. Theo API reference, trường `status` chỉ được hỗ trợ trên Amazon Nova và Anthropic Claude 3, 4, nên với model khác hãy để thông tin lỗi ngay trong `content`.
- `inputSchema` nên dùng `enum`, `minimum`, `maximum`, nhưng code tool vẫn phải kiểm tra lại input. Schema giúp model gửi đúng, không thay được bước kiểm tra phía server.
- Một nguồn dữ liệu lỗi không nên làm hỏng cả câu trả lời. Ví dụ tool dự báo vẫn chạy được mà tool cảnh báo thời tiết (`get_weather_alerts`) lỗi, thì trả lỗi cho riêng tool đó để model vẫn trả lời phần còn lại, thay vì cả request thành 500.
- Kết quả tool lấy từ nguồn bên ngoài (tin tức, trang web, email) là dữ liệu không tin cậy, có thể chứa prompt injection. Hãy ghi rõ điều này trong `description` của tool và trong system prompt.

### 4. Gọi đúng tool chưa đủ: thiết kế output cho model đọc

Đây là bài học lớn nhất của mình. Giả sử tool dự báo trả gần như nguyên dữ liệu thô: nhiệt độ ở 7 mức phân vị từ p05 đến p95. Người dùng hỏi "khả năng cao nhiệt độ mai nằm trong khoảng nào?". Model đọc nhầm, lấy p25 và p75 (khoảng 50%) rồi gọi là "khoảng 80%". Hoặc tự trừ hai số để tính chênh lệch, rồi trả ra một con số không có trong dữ liệu.

Mình đã gặp đúng kiểu lỗi đọc nhầm phân vị như vậy. Đáng sợ là các con số model nêu ra đều có thật trong kết quả tool, nên kiểm tra tự động kiểu "số trong câu trả lời có nằm trong dữ liệu không" không bắt được; phải đọc tay mới thấy.

Cách sửa theo một nguyên tắc: **thứ gì model có thể tính sai thì code tính sẵn.**

- Chỉ trả những trường thật sự cần, đặt tên nói rõ ý nghĩa: `temp_low_80`, `temp_median`, `temp_high_80` thay vì 7 phân vị trần.
- Tính sẵn các con số hay bị hỏi: `rain_prob_pct` (xác suất mưa theo %), `width_80` (độ rộng khoảng 80%).
- Đổi sẵn đơn vị, kèm đơn vị trong tên trường hoặc trong description.
- Description của tool giải thích từng trường nghĩa là gì.

Sửa output không làm model hết sai: lỗi này hết thì có thể lộ ra lỗi khác, ví dụ so sánh sai hai số âm. Tool use giải quyết chuyện lấy đúng dữ liệu, còn đọc đúng dữ liệu thì phải tiếp tục sửa ở cả output của tool lẫn prompt, và có bộ câu hỏi kiểm thử để đo lại sau mỗi lần sửa.

### 5. Checklist khi làm tool use

- Description của tool có nói rõ từng trường nghĩa là gì, đơn vị gì chưa?
- `inputSchema` có dùng `enum`, `minimum`, `maximum`? Code tool có kiểm tra lại input không?
- Có giới hạn số vòng tool và hạn chót cho cả lượt hỏi chưa?
- Có xử lý mọi giá trị `stopReason` không?
- Lỗi của tool có trả về có cấu trúc, kèm `status "error"` (nếu model hỗ trợ) không?
- `toolChoice`: `"auto"` là mặc định và mọi model có tool use đều hỗ trợ; `"any"` và `"tool"` chỉ một số model hỗ trợ, gửi sai sẽ bị `ValidationException`.
- Có ghi log tên tool đã gọi, lỗi tool và token của từng lượt để soát lại không?

### Kết luận

Với Converse API, tool use là một vòng lặp do chính code của bạn điều khiển: model đề xuất, code thực thi, kết quả quay lại model. Phần khó không nằm ở việc gọi API mà ở 2 chỗ: đặt "phanh" cho vòng lặp (số vòng, thời gian, `stopReason`) và thiết kế output của tool sao cho model khó đọc sai. Output của tool thực chất cũng là một phần của prompt.

*Lưu ý: danh sách model hỗ trợ từng tính năng (toolChoice, status, streaming kèm tool) thay đổi theo thời gian, nên kiểm tra lại tài liệu mới nhất trước khi áp dụng.*

### Tài liệu tham khảo

- [Tool use (function calling)](https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use.html)
- [Client-side tool use](https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use-client-side.html)
- [Use the Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html)
- [Converse API reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html)
- [ToolResultBlock](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_ToolResultBlock.html)
- [API Gateway REST API quotas](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-execution-service-limits-table.html)

#AWS #AmazonBedrock #ConverseAPI #ToolUse #FunctionCalling #AIAgent #AWSStudyGroup #FCAJ
