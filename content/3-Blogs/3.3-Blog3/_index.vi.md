---
title: "Blog 3"
date: 2026-11-15
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Mô hình chỉ chạy vài giây mỗi ngày: Có cần đến SageMaker Endpoint không?

bởi Trịnh Tiến Khải | ngày 15/11/2026 | đăng tại [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Bài gốc trên Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2338607513570874/)

*Bài của thành viên nhóm Trịnh Tiến Khải. Nội dung dưới đây giữ đúng câu chữ của bài đã đăng, chỉ trình bày lại bằng markdown và ghi đầy đủ đường dẫn của tài liệu tham khảo.*

Gần đây mình tìm hiểu các cách chạy inference cho mô hình học máy trên AWS, và phát hiện một điều khá thú vị: với nhiều bài toán, câu hỏi quan trọng nhất không phải "mô hình gì" mà là "mô hình được gọi như thế nào". Bài này mình ghi lại những gì đọc được, kèm một sơ đồ ví dụ có đánh số từng luồng (ảnh đính kèm) để dễ theo dõi.

**Tình huống:** một cửa hàng online muốn dự báo số đơn hàng ngày mai cho từng kho. Mỗi đêm chạy một lần sau khi chốt dữ liệu trong ngày, mô hình chạy vài giây, kết quả ghi vào database để dashboard đọc lại. Không ai cần mô hình trả lời theo từng cú click.

Phản xạ đầu tiên thường là "mô hình học máy thì lên SageMaker endpoint". Nhưng khi đọc kỹ cách tính tiền, mình thấy lựa chọn này không hợp với kiểu workload trên.

### 1. Bốn cách chạy inference trên SageMaker

- **Real-time endpoint:** tính tiền theo giờ instance trong suốt thời gian endpoint còn chạy, kể cả khi không có request nào. Hợp khi cần độ trễ thấp, lưu lượng đều, hoặc cần GPU.
- **Serverless Inference:** tự về 0 khi rảnh, đổi lại có cold start. Bộ nhớ chọn từ 1 GB đến 6 GB và không có GPU.
- **Asynchronous Inference:** request được đưa vào hàng đợi, hợp với payload lớn hoặc thời gian xử lý dài.
- **Batch Transform:** chạy theo lô trên cả tập dữ liệu, xong thì tắt.

Với job vài giây mỗi ngày, real-time endpoint gần như là trả tiền cho máy ngồi chờ. Batch Transform hay Serverless Inference dùng được, nhưng nếu mô hình nhỏ và chạy được bằng CPU thì còn một lựa chọn gọn hơn: AWS Lambda.

### 2. Vì sao là Lambda container image, không phải gói zip

- Gói zip của Lambda giới hạn 250 MB sau khi giải nén (tính cả layer). Bộ thư viện dữ liệu như pandas, pyarrow, scikit-learn cộng lại rất dễ chạm trần.
- Container image cho phép tới 10 GB (chưa nén, tính cả các layer).
- Timeout tối đa vẫn là 15 phút. Chuyển sang container không đổi giới hạn này.

Điểm mình thích nhất: cùng một image chạy được ở máy local để test trước khi đẩy lên Amazon ECR, nên "chạy được ở máy mình" và "chạy được trên Lambda" là cùng một thứ.

### 3. Đi theo 7 luồng trên sơ đồ

1. Build image, ghim base image bằng digest, tag bằng commit SHA rồi đẩy lên Amazon ECR.
2. EventBridge Scheduler chạy theo lịch cron mỗi đêm, gọi Lambda prepare theo kiểu bất đồng bộ.
3. Lambda prepare gom và làm sạch dữ liệu, ghi vào S3 theo ngày (`data/dt=…/`).
4. Prepare chạy xong, Lambda destination gọi tiếp Lambda inference. Có thể nối cả OnSuccess lẫn OnFailure: nếu một nguồn dữ liệu lỗi, inference vẫn chạy cho phần dữ liệu còn tốt.
5. Lambda inference đọc dữ liệu và trọng số mô hình trong `models/`.
6. Kết quả được ghi vào DynamoDB để dashboard hoặc API đọc lại.
7. Log và metric đi về CloudWatch: một dòng log tổng kết mỗi lần chạy và một alarm trên metric Errors là đủ để sáng hôm sau biết đêm qua chạy ra sao.

![Sơ đồ pipeline batch inference hằng đêm với Lambda container image](/images/3-Blog/blog3_figure1.jpg)

*Ảnh: sơ đồ ví dụ một pipeline batch inference hằng đêm, mình vẽ bằng draw.io dựa trên tài liệu AWS.*

Ô đỏ góc dưới sơ đồ là phương án real-time endpoint chạy 24/7, không hợp với kiểu workload này.

### 4. Những điều mình phát hiện khi đọc tài liệu

**Mô hình nhỏ không nhất thiết phải mang cả framework lên Lambda.** Với các mô hình tuyến tính hoặc mạng nhỏ, có thể train ở nơi khác, xuất trọng số ra file (ví dụ `.npz`) rồi lúc dự báo chỉ cần NumPy để nhân ma trận. Image nhẹ hơn nhiều và khởi động nhanh hơn.

**Base image không phải của AWS thì phải tự cài runtime interface client.** Base image của AWS (`public.ecr.aws/lambda/python`) có sẵn thành phần này. Nếu dùng image khác như `python:3.12-slim` thì phải cài `awslambdaric`, thiếu nó thì Lambda không gọi được handler.

```dockerfile
# Minh họa rút gọn
FROM python:3.12-slim@sha256:<digest>
WORKDIR /function
COPY requirements.txt .
RUN pip install --no-cache-dir awslambdaric -r requirements.txt
COPY src/ ./src/
ENTRYPOINT ["python", "-m", "awslambdaric"]
CMD ["src.inference.handler"]
```

Thêm một chi tiết mình phát hiện khi chạy thử: `awslambdaric` tìm module handler theo thư mục làm việc của container. Nếu thư mục làm việc không phải nơi chứa `src/`, Lambda báo `Runtime.ImportModuleError`. Vì vậy nên khai báo `WORKDIR` rõ ràng thay vì dựa vào mặc định của base image.

**Gọi bất đồng bộ thì Lambda tự thử lại.** Mặc định Lambda thử lại 2 lần khi function trả lỗi. Có thể chỉnh số lần thử lại từ 0 đến 2, và thời gian giữ event trong hàng đợi từ 60 giây đến 6 giờ. Với job inference, đặt retry bằng 0 thường hợp lý hơn: lỗi thì dừng, alarm báo, người vận hành đọc log rồi chạy lại có kiểm soát.

**Với lời gọi trực tiếp tới function, Lambda destinations chỉ áp dụng cho kiểu bất đồng bộ** (ngoài ra event source mapping của Kinesis, DynamoDB Streams và Kafka có OnFailure destination riêng). Đích có thể là Lambda khác, SQS, SNS, EventBridge, hoặc S3 (riêng S3 chỉ dùng cho OnFailure). Cái bẫy dễ gặp: `aws lambda invoke` mặc định gọi đồng bộ (`RequestResponse`), nên chạy tay thì function đầu chạy xong nhưng destination không kích hoạt. Muốn chạy lại cả chuỗi phải gọi:

```bash
aws lambda invoke --function-name <prepare> --invocation-type Event out.json
```

### 5. Chi phí

Lambda có free tier 1 triệu request và 400.000 GB-giây mỗi tháng, không hết hạn sau 12 tháng như nhiều dịch vụ khác (tính chung cho cả tài khoản). Ví dụ một function 1 GB, giả sử lần nào cũng chạy hết 120 giây, mỗi đêm một lần thì một tháng chỉ khoảng 3.600 GB-giây, chưa tới 1% free tier. ECR tính phí lưu trữ image theo dung lượng, nên đặt lifecycle policy để chỉ giữ vài image gần nhất.

### Khi nào chọn gì

- Chạy theo lô thưa, mô hình nhỏ, chạy bằng CPU, dưới 15 phút: **Lambda container image**.
- Lưu lượng thất thường, chấp nhận cold start, không cần GPU: **SageMaker Serverless Inference**.
- Payload lớn hoặc xử lý lâu: **SageMaker Asynchronous Inference**.
- Chấm điểm cả tập dữ liệu lớn theo lô: **SageMaker Batch Transform**.
- Cần độ trễ mili-giây liên tục, mô hình lớn hoặc cần GPU: **SageMaker real-time endpoint**.

### Kết luận

Chọn dịch vụ theo cách workload thật sự chạy, không theo cái nhãn "đây là bài toán ML". Với một job dự báo vài giây mỗi đêm, Lambda container image đủ chỗ cho thư viện, chạy được y hệt ở local và gần như không tốn tiền. Phần còn lại là nắm rõ kiểu gọi (đồng bộ hay bất đồng bộ), vì nhiều tính năng của Lambda như destinations hay retry chỉ áp dụng cho một kiểu.

*Lưu ý: giá và giới hạn dịch vụ có thể thay đổi, nên kiểm lại trang AWS Pricing và tài liệu mới nhất trước khi áp dụng. Đoạn Dockerfile là minh họa rút gọn.*

### Tài liệu tham khảo

- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [Create a Lambda function using a container image](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html)
- [Lambda asynchronous invocation](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html)
- [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/)
- [SageMaker Serverless Inference](https://docs.aws.amazon.com/sagemaker/latest/dg/serverless-endpoints.html)
- [Inference options in Amazon SageMaker AI](https://docs.aws.amazon.com/sagemaker/latest/dg/deploy-model-options.html)

#AWS #AWSLambda #AmazonECR #AmazonSageMaker #Serverless #MLOps #AWSStudyGroup #FCAJ
