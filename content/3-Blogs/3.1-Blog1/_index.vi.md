---
title: "Blog 1"
date: 2026-11-14
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# AgentCore Memory: Vì sao agent vừa được kể xong đã "quên"?

bởi Phan Tấn Phước | ngày 14/11/2026 | đăng tại [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Bài gốc trên Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2337675363664089/)

*Bài tự viết, lấy ý tưởng từ slide "AgentCore Memory" tại sự kiện FCAJ x FPTU: Cloud Mastery 1 (14/11/2026). Nội dung dưới đây giữ đúng câu chữ của bài đã đăng, chỉ trình bày lại bằng markdown.*

Sáng 14/11 mình tham dự FCAJ x FPTU: Cloud Mastery 1. Ở phần "The Core of Agent" có một slide về Amazon Bedrock AgentCore Memory mà mình thấy rất đáng viết riêng, vì nó giải thích một tình huống ai làm agent cũng dễ gặp. Mình vẽ lại sơ đồ đó và đánh số từng luồng dữ liệu (Hình 1) để dễ theo dõi.

**Tình huống:** bạn làm chatbot gợi ý quán ăn, dùng AgentCore Memory và bật strategy User Preference. Người dùng nhắn: "Mình ăn chay, đừng gợi ý quán thịt nướng nhé". Vài giây sau họ hỏi tiếp, agent gọi `RetrieveMemoryRecords` để tìm sở thích ăn uống... không có kết quả. Agent vẫn gợi ý quán thịt nướng.

Phản xạ đầu tiên thường là nghi strategy cấu hình sai hoặc model trích xuất kém. Nhưng nhiều khả năng nguyên nhân đơn giản hơn: long-term memory chưa kịp được tạo ra. Để hiểu vì sao, hãy đi theo 5 luồng trên sơ đồ.

### 1. Hai tầng memory và 5 luồng dữ liệu

AgentCore Memory có hai tầng:

- **Short-term memory:** lưu nguyên văn từng lượt tương tác dưới dạng event (tin nhắn, trạng thái agent), gắn với `actorId` (người dùng/agent) và `sessionId` (phiên).
- **Long-term memory:** lưu các memory record, tức thông tin đã được chắt lọc từ event, như sở thích, sự kiện quan trọng, tóm tắt phiên. Mỗi loại record do một strategy tạo ra: Semantic (sự kiện, kiến thức), User preference (sở thích), Summary (tóm tắt phiên), Episodic (kinh nghiệm theo từng "tập"), ngoài ra có thể tự định nghĩa strategy riêng.

![Sơ đồ luồng dữ liệu của AgentCore Memory](/images/3-Blog/blog1_figure1.png)

*Hình 1. Sơ đồ luồng dữ liệu của AgentCore Memory (5 luồng được đánh số), mình vẽ lại dựa trên slide "AgentCore Memory" tại sự kiện FCAJ x FPTU: Cloud Mastery 1 (14/11/2026) và tài liệu AWS*

Dữ liệu đi qua 5 luồng:

- **(1) `CreateEvent`:** agent ghi tin nhắn và trạng thái vào short-term. Đồng bộ, ghi xong là có.
- **(2) `ListEvents`:** agent đọc lại hội thoại gần đây từ short-term. Đồng bộ.
- **(3) Trích xuất tự động:** một module chạy nền đọc event mới, trích xuất (extraction) rồi hợp nhất với record đã có (consolidation), sau đó ghi vào long-term. Đây là luồng **BẤT ĐỒNG BỘ**.
- **(4) `RetrieveMemoryRecords`:** agent tìm kiếm ngữ nghĩa trong long-term theo namespace. Đồng bộ, nhưng chỉ thấy những gì luồng (3) đã xử lý xong.
- **(5) Nạp trực tiếp:** đưa dữ liệu có sẵn vào long-term mà không cần đi qua hội thoại.

### 2. Vì sao agent "quên"?

Điểm mấu chốt nằm ở luồng (3). Tài liệu AWS ghi rõ: việc trích xuất và hợp nhất long-term memory là quá trình bất đồng bộ chạy nền, có thể mất một phút hoặc hơn trước khi thông tin từ cuộc hội thoại mới có thể truy xuất được, và ứng dụng cần tính đến độ trễ này.

Trong tình huống trên, câu "ăn chay" đã vào short-term ngay ở luồng (1), nhưng luồng (3) chưa chạy xong nên luồng (4) chưa thấy record nào. Agent chỉ hỏi long-term nên không thấy gì. Đây không phải lỗi, mà là thiết kế.

![Tình huống agent "quên" trong AgentCore Memory](/images/3-Blog/blog1_figure2.png)

*Hình 2. Tình huống agent "quên": short-term ghi đồng bộ, long-term được trích xuất bất đồng bộ; cách xử lý là trong phiên dùng ListEvents (short-term), phiên sau dùng RetrieveMemoryRecords (long-term)*

Thêm một cái bẫy ít người để ý: chỉ những event được tạo **SAU** khi strategy chuyển sang trạng thái `ACTIVE` mới được đem đi trích xuất. Nếu bạn thêm strategy vào một memory đang chạy, các hội thoại cũ trước đó sẽ không tự động được xử lý.

### 3. Cách thiết kế: đọc cả hai tầng

Thay vì chỉ dựa vào luồng (4), agent nên ghép cả luồng (2) và (4) vào ngữ cảnh:

- **Trong cùng phiên:** lấy N lượt gần nhất bằng `ListEvents` (2). Thông tin người dùng vừa nói luôn có ở đây.
- **Từ các phiên trước:** dùng `RetrieveMemoryRecords` (4) theo namespace của người dùng để lấy sở thích và sự kiện đã được trích xuất.

Mã giả cho mỗi lượt hỏi:

```python
# (2) Ngữ cảnh tức thời
recent = ListEvents(memoryId, actorId, sessionId, maxResults=10)

# (4) Tri thức dài hạn đã được trích xuất
facts = RetrieveMemoryRecords(memoryId, namespace=<namespace của strategy>, searchQuery="sở thích ăn uống", topK=5)

# Gọi model với cả hai nguồn
answer = Model(prompt + recent + facts + question)

# (1) Lưu lượt mới; luồng (3) sẽ cập nhật long-term ở nền
CreateEvent(memoryId, actorId, sessionId, [user_message, answer])
```

Với cách này, sở thích vừa nói được short-term "giữ hộ" trong phiên hiện tại, còn long-term sẽ bắt kịp để dùng ở các phiên sau.

### 4. Luồng (5): khi không muốn chờ từ hội thoại

Trên slide gốc, luồng này ghi là "Batch API for long-term memory hydration". Có hai cách:

- **`BatchCreateMemoryRecords`:** tạo trực tiếp memory record với nội dung bạn tự chuẩn bị (tối đa 100 record mỗi request) và gắn vào namespace mong muốn. Phù hợp khi đã có sẵn hồ sơ khách hàng hoặc dữ liệu từ hệ thống cũ và muốn agent "biết" ngay từ đầu.
- **`IngestData`:** đẩy nội dung (hội thoại hoặc JSON như log hành vi) thẳng vào pipeline trích xuất mà không lưu thành event short-term. Cách này vẫn đi qua bước trích xuất nên vẫn bất đồng bộ; theo tài liệu, record thường xuất hiện sau vài giây đến vài phút tùy kích thước nội dung và cấu hình strategy.

### 5. Checklist khi agent "không nhớ"

- Đã chờ luồng (3) chạy xong chưa? Long-term luôn có độ trễ.
- Event có được tạo sau khi strategy `ACTIVE` không?
- Namespace lúc truy xuất ở luồng (4) có khớp với namespace mà strategy ghi vào không?
- `actorId` có nhất quán giữa các phiên của cùng một người dùng không?
- Job trích xuất có bị lỗi không? Có thể theo dõi metric `FailedExtraction` trên CloudWatch, xem job lỗi bằng `ListMemoryExtractionJobs` và chạy lại bằng `StartMemoryExtractionJob`.

### Kết luận

AgentCore Memory tách rõ "nhớ ngay" và "nhớ lâu": short-term là đường đồng bộ cho ngữ cảnh tức thời, long-term là đường bất đồng bộ cho tri thức bền vững qua nhiều phiên. Nắm được luồng (3) là bất đồng bộ thì sẽ không còn bất ngờ khi agent "quên" điều người dùng vừa nói, và thiết kế agent sẽ đúng ngay từ đầu.

*Lưu ý: AgentCore đang được cập nhật khá nhanh, nên kiểm tra lại tài liệu mới nhất trước khi áp dụng vào production.*

### Tài liệu tham khảo

- [Memory types](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory-types.html)
- [Save and retrieve insights](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/long-term-saving-and-retrieving-insights.html)
- [Built-in strategies](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/built-in-strategies.html)
- [Ingest content into long-term memory](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/long-term-ingest-data.html)
- [BatchCreateMemoryRecords API](https://docs.aws.amazon.com/bedrock-agentcore/latest/APIReference/API_BatchCreateMemoryRecords.html)
- [AWS ML Blog – Amazon Bedrock AgentCore Memory](https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-agentcore-memory-building-context-aware-agents/)

#AWS #AmazonBedrock #BedrockAgentCore #AgentCoreMemory #AIAgent #AWSStudyGroup #FCAJ
