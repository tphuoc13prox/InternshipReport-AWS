---
title: "Blog 2"
date: 2026-11-16
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Khách hàng đặt hàng thành công nhưng hệ thống kho không nhận được event: Transactional Outbox giải quyết thế nào trên AWS?

bởi Huỳnh Ngọc Tuấn | ngày 16/11/2026 | đăng tại [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Bài gốc trên Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338993156865643/)

*Bài của thành viên nhóm Huỳnh Ngọc Tuấn. Nội dung dưới đây giữ đúng câu chữ của bài đã đăng, chỉ trình bày lại bằng markdown và ghi đầy đủ đường dẫn của tài liệu tham khảo.*

Khi thiết kế một hệ thống bán hàng trên AWS, có một tình huống khá nguy hiểm mà mình nghĩ nhiều người có thể chưa để ý: khách hàng đặt hàng thành công, database đã lưu đơn hàng nhưng hệ thống kho lại không nhận được thông báo để chuẩn bị hàng.

Thoạt nhìn, đây có vẻ chỉ là lỗi gửi message. Nhưng thực tế, nó liên quan đến một vấn đề kinh điển trong kiến trúc Microservices: **Dual Write Problem**.

Trong bài này, mình tìm hiểu cách giải quyết bằng Transactional Outbox Pattern, kết hợp Amazon DynamoDB Streams và Amazon EventBridge Pipes. Đây là một cách xây dựng hệ thống xử lý event theo hướng Event-Driven Architecture trên AWS.

### 1. Vấn đề: database đã lưu nhưng event bị mất

Giả sử chúng ta có một website bán hàng với quy trình:

1. Khách hàng gửi yêu cầu đặt hàng.
2. Order Service lưu đơn hàng vào database.
3. Order Service gửi event `OrderCreated` để thông báo cho Inventory Service.
4. Inventory Service nhận event và cập nhật số lượng hàng tồn kho.

Một cách triển khai thông thường có thể viết như sau:

```python
def create_order(order):
    save_order_to_database(order)
    publish_event({
        "eventType": "OrderCreated",
        "orderId": order["orderId"]
    })
```

Nhìn qua thì không có gì bất thường. Nhưng hãy tưởng tượng:

- Database lưu đơn hàng thành công.
- Ngay sau đó, ứng dụng gặp lỗi hoặc mất kết nối trước khi gửi event.
- Inventory Service không nhận được thông báo.

Kết quả: đơn hàng tồn tại trong database nhưng hệ thống kho không biết đơn hàng đó vừa được tạo.

Một tình huống khác cũng nguy hiểm không kém: event được gửi thành công nhưng thao tác lưu database thất bại. Khi đó, hệ thống kho có thể xử lý một đơn hàng không tồn tại.

Đây chính là Dual Write Problem: ứng dụng phải ghi vào hai hệ thống độc lập, nhưng không có một transaction chung đảm bảo cả hai thao tác cùng thành công hoặc cùng thất bại.

### 2. Transactional Outbox Pattern giải quyết như thế nào?

Ý tưởng của Transactional Outbox khá đơn giản: thay vì ghi database rồi gửi event trực tiếp, chúng ta lưu cả dữ liệu nghiệp vụ và event cần phát đi trong cùng một transaction database.

Ví dụ, khi khách hàng đặt hàng, hệ thống ghi hai bản ghi:

Bản ghi Order:

```json
{
  "PK": "ORDER#1001",
  "SK": "DETAIL",
  "status": "CREATED",
  "total": 1500000
}
```

Bản ghi Outbox Event:

```json
{
  "PK": "OUTBOX#evt-1001",
  "SK": "EVENT",
  "eventType": "OrderCreated",
  "orderId": "1001",
  "status": "PENDING"
}
```

Hai thao tác này được thực hiện trong cùng một DynamoDB transaction bằng `TransactWriteItems`.

Điều quan trọng là:

- Nếu ghi Order thành công và ghi Outbox thất bại, toàn bộ transaction bị rollback.
- Nếu transaction thành công, cả Order và Outbox Event đều được lưu.
- Nếu transaction thất bại, không có event hợp lệ nào được tạo ra từ lần đặt hàng đó.

Như vậy, ứng dụng không còn phải thực hiện hai thao tác độc lập là ghi database và gửi message trong cùng một luồng xử lý. Thay vào đó, việc phát event được thực hiện bất đồng bộ sau khi transaction đã commit.

### 3. Triển khai trên AWS với 7 thành phần

Để minh họa, mình sử dụng kiến trúc gồm các dịch vụ sau:

- **Amazon API Gateway:** Tiếp nhận request đặt hàng.
- **AWS Lambda – Order Service:** Xử lý nghiệp vụ và tạo transaction.
- **Amazon DynamoDB:** Lưu Order và Outbox Event.
- **DynamoDB Streams:** Ghi nhận các thay đổi trên DynamoDB.
- **Amazon EventBridge Pipes:** Đọc, lọc và chuyển tiếp event từ Stream.
- **Amazon EventBridge:** Định tuyến event đến các consumer phù hợp.
- **Amazon SQS và Lambda:** Tiếp nhận và xử lý công việc bất đồng bộ.

Luồng hoạt động cụ thể:

**(1) Khách hàng gửi request.** Client gửi HTTP POST đến API Gateway với thông tin đơn hàng.

**(2) Lambda xử lý và ghi transaction.** API Gateway gọi Order Service Lambda. Lambda sử dụng `TransactWriteItems` để ghi đồng thời Order và Outbox Event vào DynamoDB.

**(3) DynamoDB Streams ghi nhận thay đổi.** Sau khi transaction thành công, các thay đổi của item được ghi nhận trong DynamoDB Streams. Stream cung cấp một chuỗi thay đổi theo thứ tự, cho phép các thành phần phía sau đọc và xử lý dữ liệu mới.

**(4) EventBridge Pipes tiếp nhận event.** EventBridge Pipes kết nối trực tiếp với DynamoDB Streams. Chúng ta có thể cấu hình Filter Criteria để chỉ chuyển tiếp những bản ghi Outbox cần xử lý, thay vì đưa tất cả thay đổi trong bảng vào event bus. Pipes cũng hỗ trợ biến đổi dữ liệu trước khi gửi đến target.

**(5) Event được đưa vào EventBridge.** EventBridge tiếp nhận event và sử dụng Event Rules để định tuyến đến các dịch vụ quan tâm. Ví dụ:

- `OrderCreated` → Inventory Service.
- `OrderPaid` → Notification Service.
- `OrderCancelled` → Order Management Service.

**(6) SQS tiếp nhận công việc.** Các event phù hợp được chuyển vào SQS Queue. SQS đóng vai trò buffer, giúp consumer xử lý công việc độc lập với tốc độ phát sinh request từ phía khách hàng.

**(7) Lambda xử lý event.** Inventory Lambda đọc message từ SQS và cập nhật số lượng hàng tồn kho. Nếu xử lý thất bại nhiều lần theo cấu hình retry, message có thể được chuyển vào Dead-Letter Queue để kiểm tra và xử lý lại.

![Kiến trúc Transactional Outbox với DynamoDB Streams, EventBridge Pipes và EventBridge](/images/3-Blog/blog2_figure1.jpg)

*Hình: Kiến trúc Transactional Outbox sử dụng Amazon DynamoDB Streams, EventBridge Pipes và Amazon EventBridge để phân phối sự kiện đến nhiều consumer. Nguồn: AWS Compute Blog*

### 4. EventBridge Pipes có gì đặc biệt?

Một điểm mình thấy khá thú vị là trước đây, nếu muốn đọc dữ liệu từ DynamoDB Streams rồi chuyển tiếp sang hệ thống khác, chúng ta thường phải viết thêm một Lambda trung gian để poll Stream, xử lý dữ liệu và gửi event.

Với Amazon EventBridge Pipes, AWS cung cấp một thành phần managed để kết nối trực tiếp source và target. Một số khả năng đáng chú ý:

- **Filtering:** Chỉ chuyển tiếp những event thỏa mãn điều kiện.
- **Transformation:** Biến đổi cấu trúc event trước khi gửi đến target.
- **Retry:** Cấu hình cơ chế thử lại khi xử lý gặp lỗi.
- **Dead-letter queue:** Có thể cấu hình SQS DLQ để lưu các event không thể xử lý sau các lần thử lại.

Điều này giúp giảm lượng code trung gian mà đội phát triển phải tự quản lý.

Tuy nhiên, Pipes không làm cho toàn bộ hệ thống trở thành một transaction duy nhất. Nó chỉ giúp kết nối và chuyển tiếp event giữa các thành phần.

### 5. Có Transactional Outbox rồi thì event có được gửi đúng một lần không?

Đây là chi tiết rất quan trọng. Câu trả lời là: **Không nên mặc định event chỉ được xử lý đúng một lần.**

DynamoDB Streams, EventBridge và SQS đều có những cơ chế retry hoặc giao nhận riêng. Trong một số tình huống, cùng một event có thể được xử lý lại. Ví dụ:

1. Inventory Lambda nhận event `OrderCreated`.
2. Lambda cập nhật số lượng hàng tồn kho thành công.
3. Nhưng trước khi hệ thống xác nhận hoàn tất xử lý message, một lỗi xảy ra.
4. Message được gửi lại.
5. Lambda có thể thực hiện lại nghiệp vụ cập nhật tồn kho.

Nếu không có cơ chế bảo vệ, số lượng hàng có thể bị trừ hai lần.

Giải pháp là **Idempotency**. Mỗi event nên có một `eventId` duy nhất. Consumer lưu lại các event đã xử lý và kiểm tra ID trước khi thực hiện nghiệp vụ. Ví dụ:

```python
def process_event(event):
    event_id = event["eventId"]

    if already_processed(event_id):
        return

    process_business_logic(event)
    mark_as_processed(event_id)
```

Đây chỉ là ví dụ minh họa. Trong hệ thống thực tế, việc kiểm tra, thực hiện nghiệp vụ và đánh dấu event đã xử lý cần được thiết kế phù hợp để tránh race condition. Với DynamoDB, có thể sử dụng conditional write hoặc transaction để bảo vệ thao tác này.

Một điểm khác cần lưu ý là DynamoDB Streams lưu giữ record trong tối đa 24 giờ. Vì vậy, hệ thống cần giám sát tiến độ xử lý và có phương án phục hồi phù hợp nếu consumer bị gián đoạn lâu.

### 6. Khi nào nên sử dụng kiến trúc này?

Transactional Outbox kết hợp DynamoDB Streams và EventBridge Pipes phù hợp với những hệ thống:

- Có nhiều Microservices cần nhận thông báo khi dữ liệu thay đổi.
- Muốn tách biệt thao tác ghi database và xử lý event.
- Cần giảm nguy cơ mất thông báo khi ứng dụng gặp lỗi sau khi lưu dữ liệu.
- Muốn xử lý nghiệp vụ bất đồng bộ.
- Cần mở rộng thêm consumer mà không phải thay đổi trực tiếp Order Service.

Tuy nhiên, không phải hệ thống nào cũng cần triển khai đầy đủ các thành phần trên. Nếu ứng dụng nhỏ, chỉ có một database và một consumer đơn giản, việc bổ sung quá nhiều dịch vụ có thể làm tăng chi phí và độ phức tạp vận hành.

Ngoài ra, nếu sử dụng Amazon RDS hoặc Aurora, có thể triển khai Outbox Table theo mô hình relational database và dùng một publisher riêng để đọc outbox rồi gửi message đến SQS.

Việc lựa chọn phụ thuộc vào database hiện tại, yêu cầu transaction, lưu lượng event và khả năng vận hành của đội ngũ.

### Kết luận

Điều mình rút ra khi tìm hiểu Transactional Outbox Pattern là: trong một hệ thống phân tán, không nên mặc định rằng việc lưu dữ liệu thành công đồng nghĩa với việc các service khác đã nhận được thông báo.

Thay vì cố gắng thực hiện hai thao tác độc lập trong cùng một request, chúng ta có thể lưu dữ liệu nghiệp vụ và event trong một transaction, sau đó sử dụng cơ chế CDC để phát event bất đồng bộ.

Với AWS, sự kết hợp giữa DynamoDB, DynamoDB Streams, EventBridge Pipes, EventBridge và SQS tạo ra một kiến trúc khá linh hoạt cho các ứng dụng Event-Driven.

Tất nhiên, Transactional Outbox không loại bỏ hoàn toàn mọi lỗi trong hệ thống. Duplicate delivery, retry, thứ tự event và idempotency vẫn là những vấn đề cần được thiết kế cẩn thận. Nhưng đây là một pattern đáng tìm hiểu nếu muốn xây dựng các hệ thống Microservices có khả năng xử lý lỗi tốt hơn.

### Tài liệu tham khảo – AWS Official

- [Transactional Outbox Pattern – AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)
- [Implementing the Transactional Outbox Pattern with Amazon EventBridge Pipes – AWS Compute Blog](https://aws.amazon.com/blogs/compute/implementing-the-transactional-outbox-pattern-with-amazon-eventbridge-pipes/)
- [DynamoDB Transactions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html)
- [DynamoDB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)
- [Amazon EventBridge Pipes](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-pipes.html)
- [Amazon SQS Dead-Letter Queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [AWS Sample – Transactional Outbox Pattern](https://github.com/aws-samples/transactional-outbox-pattern)

#AWS #AmazonDynamoDB #DynamoDBStreams #AmazonEventBridge #EventBridgePipes #AmazonSQS #Microservices #EventDrivenArchitecture #CloudArchitecture #AWSStudyGroup #FCAJ
