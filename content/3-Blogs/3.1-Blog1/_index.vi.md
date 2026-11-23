---
title: "Blog 1"
date: 2026-11-17
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Prompt caching cho chatbot có tool use: Đặt cachePoint đúng chỗ để giảm tới ~90% chi phí phần prompt lặp lại trên Amazon Bedrock

bởi Phan Tấn Phước | ngày 17/11/2026 | đăng tại [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Bài gốc trên Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2340410270057265/)

*Bài tự viết, tổng hợp từ tài liệu AWS; các con số trong bài là ví dụ minh họa. Nội dung dưới đây giữ đúng câu chữ của bài đã đăng, chỉ trình bày lại bằng markdown.*

Ở bài trước về tool use với Converse API, mình có nhắc: Converse không lưu hội thoại giúp bạn, nên mỗi vòng tool là một lần gọi model và phải gửi lại toàn bộ định nghĩa tool, system prompt và lịch sử. Một câu hỏi cần 2 vòng tool là 3 lần gọi, và phần "giống hệt nhau" đó bị tính tiền 3 lần. Prompt caching là cách để trả tiền phần lặp lại đó rẻ hơn nhiều, đồng thời giảm độ trễ. Bài này mình tổng hợp theo tài liệu AWS, kèm cách sắp xếp prompt để cache thật sự "trúng". Các con số trong bài là ví dụ minh họa, sơ đồ ở Hình 1.

### 1. Prompt caching hoạt động thế nào?

Bedrock cache theo tiền tố (prefix) của prompt. Bạn đặt các mốc cache (cache checkpoint) bằng khối `cachePoint`, phần nội dung đứng trước mốc được lưu lại; request sau có cùng tiền tố đó sẽ đọc từ cache thay vì xử lý lại.

- Có 2 kiểu: implicit (Bedrock tự thử dùng lại tiền tố, không cần cấu hình, nhưng chỉ là best effort) và explicit (bạn tự đặt `cachePoint`). Với Claude trên Bedrock, phải dùng explicit: request không có cache control thì không được cache.
- Trong Converse, `cachePoint` đặt được ở 3 chỗ: `tools`, `system` và `messages`. Thứ tự xử lý là tools → system → messages, và chúng nối tiếp nhau: sửa phần tools thì cache của system và messages phía sau cũng mất.
- Claude cho tối đa 4 mốc cache mỗi request.
- Cache có TTL, mặc định 5 phút và được làm mới mỗi lần đọc trúng. Một số model hỗ trợ TTL 1 giờ (`"ttl": "1h"`).

Cú pháp rất ngắn, chỉ là thêm một khối vào sau phần muốn cache:

```json
{"cachePoint": {"type": "default"}}
```

![Sơ đồ prompt caching trong vòng lặp tool use với Converse API](/images/3-Blog/blog1_figure1.png)

*Hình 1. Sơ đồ cách đặt cachePoint trong vòng lặp tool use và chi phí minh họa, mình tự vẽ dựa trên tài liệu AWS*

### 2. Tiền ở đâu ra?

Token đầu vào giờ có 3 mức giá: token ghi vào cache, token đọc từ cache, và token thường. Theo bảng giá Bedrock, với nhiều model Claude: ghi cache 5 phút bằng 1,25 lần giá input thường, ghi cache 1 giờ bằng 2 lần, còn đọc cache chỉ bằng 0,1 lần (có model còn thấp hơn). Token đầu ra không đổi giá.

Ví dụ minh họa: chatbot có định nghĩa tool + system prompt dài 5.000 token, mỗi câu hỏi trung bình 3 lần gọi model (2 vòng tool). Chỉ tính riêng phần 5.000 token lặp lại, quy ra "đơn vị = giá 1 token input thường":

- Không cache: 3 × 5.000 = 15.000 đơn vị.
- Có cache, cache đang trống (câu hỏi đầu tiên): ghi 5.000 × 1,25 + đọc 2 × 5.000 × 0,1 = 7.250 đơn vị, giảm khoảng 52%.
- Có cache, cache đang "ấm" (có request khác trong 5 phút trước đó): đọc 3 × 5.000 × 0,1 = 1.500 đơn vị, giảm 90%.

Chatbot càng nhiều vòng tool, system prompt càng dài, người dùng hỏi càng đều tay thì càng lợi. Ngoài tiền, tài liệu còn nêu 2 lợi ích nữa: giảm độ trễ vì model không phải xử lý lại phần đã cache, và token đọc từ cache không bị trừ vào rate limit.

### 3. Sáng kiến: sắp xếp prompt để cache trúng

Bật `cachePoint` thì dễ, làm cho nó trúng mới khó. Vì cache khớp theo tiền tố chính xác, chỉ cần một ký tự ở đầu thay đổi là cả phần sau trượt. Mình rút ra mấy nguyên tắc:

- **Tĩnh trước, động sau.** Thứ tự lý tưởng: định nghĩa tool → system prompt cố định → lịch sử hội thoại → câu hỏi mới.
- **Đưa phần động ra khỏi system prompt.** Lỗi rất hay gặp là chèn "Hôm nay là 17/11/2026, 10:32" hay tên người dùng vào đầu system prompt: mỗi request một khác, cache không bao giờ trúng. Hãy đưa những thông tin này xuống message của người dùng, sau mốc cache.
- **Giữ danh sách tool cố định và đúng thứ tự.** Nếu code dựng `toolConfig` từ một dict/set mà thứ tự không ổn định, hoặc mỗi nhóm người dùng một bộ tool khác nhau, tiền tố sẽ khác nhau và cache trượt. Sắp xếp tool theo tên trước khi gửi là đủ.
- **Dùng mốc cache "di động" trong vòng lặp tool.** Ngoài mốc sau tools/system, đặt thêm một `cachePoint` ở cuối message mới nhất (message chứa `toolResult`). Vòng sau sẽ đọc lại được cả lịch sử của vòng trước thay vì chỉ phần tools/system. Lưu ý tài liệu nói Claude chỉ dò ngược khoảng 20 content block trước mỗi mốc, nên hội thoại dài thì nên có nhiều mốc.
- **Tách mốc theo tần suất thay đổi.** Ví dụ một mốc sau tools, một mốc sau system prompt: khi bạn sửa system prompt, phần tools vẫn được đọc từ cache.

Với các model Claude mới, còn có "automatic caching": thêm một trường `cache_control` ở cấp request (qua `additionalModelRequestFields` trong Converse), Claude tự đặt mốc ở block cuối và dời dần theo hội thoại. Cách này đơn giản hơn nhưng vẫn phải tuân thủ nguyên tắc "tĩnh trước, động sau".

### 4. Những bẫy cần biết

- **Ngưỡng tối thiểu:** tiền tố trước mỗi mốc phải đủ số token tối thiểu của model, hiện từ 512 đến 4.096 token tùy model (ví dụ Claude Haiku 4.5 cần 4.096, Claude Sonnet 4.6 cần 1.024). Chưa đủ ngưỡng thì request vẫn chạy bình thường nhưng KHÔNG cache gì cả, và không báo lỗi. Ngưỡng tính dồn trên cả tools + system + messages.
- **Không đảm bảo trúng:** cache nằm trên server đã xử lý request. Request rơi vào server khác (ví dụ Region khác khi dùng cross-Region inference), hoặc gửi trùng nhau trước khi request đầu kịp trả lời, sẽ ghi entry mới thay vì đọc. Tài liệu cũng nói lúc tải cao, cross-Region inference có thể làm tăng số lần ghi cache.
- **Ghi đắt hơn đọc:** nếu traffic thưa (người dùng hỏi cách nhau hơn 5 phút), cache hết hạn liên tục và bạn trả phí ghi 1,25 lần mà không đọc lại được. Khi đó cân nhắc TTL 1 giờ (ghi 2 lần giá) hoặc không cache.
- Chỉ dùng được với on-demand, không hỗ trợ batch inference.
- Đặt `ttl` cho model chỉ hỗ trợ 5 phút có thể bị `ValidationException`.

### 5. Đo để biết có hiệu quả không

Response của Converse có thêm `cacheReadInputTokens` và `cacheWriteInputTokens`. Lưu ý khi bật cache, `inputTokens` chỉ còn là phần KHÔNG cache, nên:

```text
tổng token đầu vào = inputTokens + cacheReadInputTokens + cacheWriteInputTokens
```

Mình khuyên log 3 con số này cho mỗi lần gọi và theo dõi tỉ lệ đọc cache = cacheRead / tổng token đầu vào. Nếu `cacheWriteInputTokens` luôn bằng 0 thì nhiều khả năng tiền tố chưa đủ ngưỡng; nếu lần nào cũng ghi mà không đọc thì có phần động đang nằm trước mốc cache.

### Checklist

- Model của bạn có hỗ trợ prompt caching không, ngưỡng tối thiểu bao nhiêu token, TTL nào?
- Tools và system prompt có cố định, đúng thứ tự giữa các request không?
- Đã đưa ngày giờ, tên người dùng, dữ liệu theo phiên ra khỏi phần trước mốc cache chưa?
- Đã có mốc cache cuối message mới nhất trong vòng lặp tool chưa?
- Traffic có đủ dày để cache 5 phút còn "ấm" không? Nếu không, TTL 1 giờ có đáng không?
- Đã log `cacheReadInputTokens` / `cacheWriteInputTokens` để đo tỉ lệ trúng chưa?

### Kết luận

Với chatbot có tool use, phần prompt lặp lại thường lớn hơn nhiều so với câu hỏi mới của người dùng. Prompt caching biến phần lặp lại đó thành token "đọc cache" với giá chỉ khoảng một phần mười, nhưng chỉ khi bạn sắp xếp prompt theo nguyên tắc tĩnh trước, động sau và đo lại bằng số liệu cache trong response. Đây là một trong những cách tối ưu có tỉ lệ "công sức bỏ ra / tiền tiết kiệm" tốt nhất mà mình biết trên Bedrock.

*Lưu ý: danh sách model hỗ trợ, ngưỡng token, TTL và giá cache thay đổi khá thường xuyên, nên kiểm tra lại tài liệu và bảng giá mới nhất trước khi áp dụng.*

### Tài liệu tham khảo

- [Prompt caching for faster model inference](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html)
- [CachePointBlock (API reference)](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_CachePointBlock.html)
- [Use the Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html)
- [Tool use (function calling)](https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use.html)
- [Amazon Bedrock pricing](https://aws.amazon.com/bedrock/pricing/)

#AWS #AmazonBedrock #PromptCaching #ConverseAPI #ToolUse #CostOptimization #GenAI #AWSStudyGroup #FCAJ
