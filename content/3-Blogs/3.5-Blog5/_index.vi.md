---
title: "Blog 5"
date: 2026-11-15
weight: 5
chapter: false
pre: " <b> 3.5. </b> "
---

# Mỗi tối chạy một việc rồi xử lý tiếp: Có cần một server chạy 24/7 hay Step Functions không?

bởi Châu Trần Mạnh Tiến | ngày 15/11/2026 | đăng tại [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Bài gốc trên Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338839130214379/)

*Bài của thành viên nhóm Châu Trần Mạnh Tiến. Nội dung dưới đây giữ đúng câu chữ của bài đã đăng, chỉ trình bày lại bằng markdown và ghi đầy đủ đường dẫn của tài liệu tham khảo.*

Có một kiểu bài toán rất hay gặp: mỗi ngày vào một giờ cố định, gọi một API bên ngoài để lấy dữ liệu, rồi xử lý dữ liệu đó và lưu kết quả lại. Cách quen thuộc nhất là thuê một máy chủ và đặt cron. Nhưng nếu mỗi việc chỉ chạy vài chục giây mỗi ngày, để một máy chạy suốt 24 giờ thì khá phí, lại phải tự vá lỗi, tự theo dõi máy đó.

Khi tìm hiểu và thử dựng luồng này theo kiểu serverless trên AWS, mình thấy chỉ cần ghép vài dịch vụ là đủ: Amazon EventBridge Scheduler, AWS Lambda và Lambda Destinations. Bài này mình tổng hợp lại cách ghép, kèm những chỗ dễ sai mà mình đã gặp.

### Sơ đồ tổng quan

```text
EventBridge Scheduler (giờ cố định mỗi ngày)
        │ gọi bất đồng bộ
        ▼
Lambda "lấy dữ liệu" ──► lưu vào S3
        │ Lambda Destinations (OnSuccess / OnFailure)
        ▼
Lambda "xử lý" ──► ghi kết quả vào DynamoDB

CloudWatch Alarm (metric Errors) ──► SNS ──► email
```

![Sơ đồ kiến trúc EventBridge Scheduler, Lambda và Lambda Destinations](/images/3-Blog/blog5_figure1.jpg)

*Ảnh đính kèm bài gốc: sơ đồ kiến trúc của luồng.*

### 1. EventBridge Scheduler: đặt lịch thay cho cron

EventBridge Scheduler cho phép tạo lịch theo cron hoặc theo khoảng thời gian, chọn được múi giờ cho từng lịch, có lịch chạy một lần, và có cửa sổ thời gian linh hoạt (flexible time window) khi không cần chạy đúng từng phút.

Điểm mình thấy cần để ý là retry policy. Theo tài liệu AWS, số lần retry tối đa đặt được từ 0 tới 185, mặc định là 185, và thời gian giữ sự kiện để thử lại từ 60 giây tới 24 giờ. Sự kiện gửi không thành công có thể đưa vào một hàng đợi SQS làm dead-letter queue (DLQ).

Scheduler cần một IAM role để gọi Lambda. Role này nên chỉ có quyền `lambda:InvokeFunction` trên đúng một function. Trust policy nên thêm điều kiện `aws:SourceAccount` và `aws:SourceArn`, để chỉ lịch trong đúng tài khoản và đúng schedule group mới dùng được role. Đây là cách AWS khuyến nghị để tránh lỗi "confused deputy".

### 2. Lambda từ container image khi thư viện quá nặng

Nếu code cần các thư viện lớn như pandas hay các thư viện machine learning, gói zip của Lambda dễ vượt giới hạn 250 MB sau khi giải nén. Khi đó có thể đóng gói thành container image (tối đa 10 GB), đẩy lên Amazon ECR và cho Lambda chạy trực tiếp từ image.

Đổi lại, image lớn thì cold start chậm hơn. Lần chạy đầu tiên mình từng gặp giai đoạn init của Lambda vượt 10 giây vì phải import thư viện nặng. Lambda tự init lại nên lượt chạy vẫn thành công, nhưng nên theo dõi dòng `INIT_REPORT` và `REPORT` trong CloudWatch Logs.

Mỗi lần build lại sẽ có thêm một image trong ECR. Nên đặt lifecycle rule (ví dụ giữ 5 image mới nhất) để không tốn tiền lưu trữ image cũ.

### 3. Lambda Destinations: nối hai bước mà không cần Step Functions

Với lời gọi bất đồng bộ, Lambda cho phép cấu hình nơi nhận kết quả khi chạy thành công (OnSuccess) và khi thất bại sau khi đã hết retry (OnFailure). Đích có thể là một Lambda khác, hàng đợi SQS, topic SNS, event bus của EventBridge, và với OnFailure thì còn có thể là S3 bucket.

Theo tài liệu, khi gọi bất đồng bộ, Lambda mặc định retry 2 lần nếu function báo lỗi (đặt được từ 0 tới 2), và giữ sự kiện trong hàng đợi tối đa 6 giờ.

Với luồng chỉ có hai bước, cho OnSuccess trỏ sang Lambda "xử lý" là đủ. Nếu bước thứ nhất lỗi một phần nhưng vẫn có dữ liệu dùng được, có thể cho cả OnFailure trỏ sang bước thứ hai.

Ví dụ bằng AWS CDK (Python):

```python
fetch_fn = lambda_.DockerImageFunction(
    self, "FetchFunction",
    code=lambda_.DockerImageCode.from_ecr(repo, tag_or_digest="v1"),
    memory_size=1024,
    timeout=Duration.seconds(300),
    retry_attempts=1,
    on_success=destinations.LambdaDestination(process_fn),
    on_failure=destinations.LambdaDestination(process_fn),
)
```

### 4. Giám sát: biết ngay khi có lỗi

Một luồng chạy tự động mà không có cảnh báo thì rất dễ hỏng âm thầm. Cách đơn giản là tạo CloudWatch Alarm trên metric Errors của từng Lambda, ngưỡng từ 1 lỗi trở lên, rồi gửi qua SNS về email.

### Những điều cần lưu ý

- **Destinations chỉ chạy khi gọi bất đồng bộ.** Scheduler gọi bất đồng bộ nên luồng chạy đúng. Nhưng khi tự thử bằng `aws lambda invoke` mặc định (đồng bộ) thì bước thứ hai sẽ không được gọi. Muốn thử cả luồng thì thêm `--invocation-type Event`.
- **Chỉ để retry ở một tầng.** Scheduler mặc định retry rất nhiều lần, Lambda cũng có retry riêng. Mình đặt retry của Scheduler bằng 0 và để retry ở Lambda, cho dễ đoán số lần gọi API bên ngoài và dễ đọc log.
- **Lambda chạy tối đa 15 phút mỗi lần gọi.** Việc nào lâu hơn thì nên dùng ECS, AWS Batch hoặc Step Functions để chia nhỏ.
- **Khi nào nên chuyển sang Step Functions:** khi luồng có nhiều bước, có rẽ nhánh, cần chờ, cần chạy song song, hoặc cần nhìn trạng thái từng bước trên một màn hình. Destinations hợp với chuỗi ngắn, đơn giản.
- **Nếu viết bằng CDK, luôn `cdk diff` trước `cdk deploy`.** Đọc kỹ các dòng `[-]` (tài nguyên sắp bị xóa) trước khi xác nhận.

### Kết luận

Với kiểu việc "mỗi ngày chạy một lần, vài chục giây, rồi làm tiếp một bước", combo EventBridge Scheduler + Lambda + Lambda Destinations đủ dùng mà không cần server chạy cả ngày hay một state machine. Bạn chỉ trả tiền cho thời gian chạy thật, và có cảnh báo khi có lỗi.

Đây là những gì mình tìm hiểu và tự thử trong quá trình học, chắc chắn còn thiếu sót. Mong mọi người góp ý, nhất là các anh chị đã chạy kiểu luồng này lâu dài trên production.

### Tài liệu tham khảo

- [Amazon EventBridge Scheduler – User Guide](https://docs.aws.amazon.com/scheduler/latest/UserGuide/what-is-scheduler.html)
- [RetryPolicy – EventBridge Scheduler API Reference](https://docs.aws.amazon.com/scheduler/latest/APIReference/API_RetryPolicy.html)
- [Configuring error handling for Lambda asynchronous invocations](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async-error-handling.html)
- [Lambda Destinations cho lời gọi bất đồng bộ](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async-retain-records.html)
- [Tạo Lambda function bằng container image](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html)
- [The confused deputy problem – IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html)

#AWS #AWSStudyGroupVN #FCAJ #Serverless #AWSLambda #EventBridge #AWSCDK
