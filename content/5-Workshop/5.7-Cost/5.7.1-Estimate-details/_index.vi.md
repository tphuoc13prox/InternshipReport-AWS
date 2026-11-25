---
title: "Phụ lục: chi tiết dự toán"
date: 2026-11-25
weight: 1
chapter: false
pre: " <b> 5.7.1. </b> "
---
Trang này ghi đủ giả định và 34 dòng của dự toán chi phí do Nguyễn Tiến Long lập ngày 24/11/2026 (phần tóm tắt ở mục [5.7](../)). Dự toán áp dụng cho một môi trường `cpaws-dev-*` tại Singapore. Số liệu quan sát trong tháng 11 gồm cả phát triển, kiểm thử và demo, vì hệ thống chưa chạy production trọn một tháng. Đây là chi phí dự toán, không phải hóa đơn hay hạn mức chi tiêu.

Bảng Pricing Calculator được xuất lúc 23:57 ngày 24/11/2026 (UTC+7), mốc giá 23:33 ngày 23/11/2026, chế độ `BEFORE_DISCOUNTS`, 34 dòng, tất cả ở `ap-southeast-1`. Bedrock và EventBridge Scheduler được lập bằng Pricing Calculator public (link ở mục 5.7); SSM Parameter Store tính theo bảng giá Standard. File CSV xuất từ Console có Account ID đầy đủ nên không công bố.

#### 1. Phương pháp

Mỗi số liệu được xếp vào một trong bốn loại: **quan sát** (usage và metric đã có), **cấu hình** (tài nguyên và mã nguồn), **giả định** (khối lượng trong tương lai) và **dự toán** (giá từ Calculator). Các dòng mới có Historical cost = 0 không được hiểu là hóa đơn thực tế bằng 0. Một số số liệu Cost Explorer ở cấp tài khoản hoặc region chưa tách riêng được theo tag CPAWS, nên chỉ dùng làm số đại diện và có ghi chú.

Usage đo trong d ngày được quy về tháng 30 ngày bằng `U_month = U_observed × 30 / d`, làm tròn theo đơn vị. Với tài nguyên tồn tại cả tháng (MAU, alarm, dung lượng lưu trữ) hoặc lịch cron đã biết, dự toán dùng cấu hình và mức hiện tại thay vì nhân máy móc. Chi phí năm bằng 12 lần chi phí tháng, không giả định tăng trưởng, chưa tính thuế và thay đổi giá.

##### 1.1. Tải ứng dụng

| Thông số | Cơ sở | Mỗi tháng |
|---|---|---:|
| Cognito Essentials MAU | Cost Explorer: 3 trong tháng 10, 4 từ 01–23/11; giả định 5 người hoạt động | 5 |
| Forecast API | Giả định 60 request/ngày, không phải số đo traffic | 1.800 |
| Accuracy API | Giả định 20 request/ngày | 600 |
| Chat API | Giả định 10 câu/ngày | 300 |
| API Gateway | 1.800 + 600 + 300; chưa cộng riêng health, preflight và retry | 2.700 |
| Scheduler | 2 lịch × 5 ngày/tuần, làm tròn 22 lượt mỗi lịch | 44 |
| Lambda job | 22 ingest + 22 predict + 22 accuracy-job; mỗi ingest kéo theo một predict | 66 |
| Lambda tổng | 2.700 API + 66 job | 2.766 |
| Bedrock | 7 câu chat thành công quan sát ngày 22–23/11, gồm 13 lần gọi model | khoảng 557 lần gọi |
| Amplify request | 679 request trong 2 ngày × 15; chỉ dùng để ước lượng tải hosting | 10.185 |

Mã nguồn giới hạn chatbot ở 20 câu/người/ngày và 100 câu/ngày cho cả hệ thống. Mức 300 câu/tháng của dự toán thấp hơn nhiều so với mức trần 3.000 câu/tháng. Giới hạn lượt hỏi cũng không phải giới hạn tiền hay token.

##### 1.2. Lambda: đo thực tế và quy về tháng

Số đo lấy từ Lambda và CloudWatch trong 17–23/11/2026. Tất cả hàm chạy x86_64 với ephemeral storage 512 MB. Thời gian chạy là số quan sát, không phải timeout. Compute = `bộ nhớ (GB) × thời gian chạy (giây) × số lượt gọi`.

| Hàm | RAM (MB) | Thời gian chạy TB (ms) | Lượt gọi trong 7 ngày | Thời gian dùng để dự toán (s) | Lượt/tháng | GB-giây/tháng |
|---|---:|---:|---:|---:|---:|---:|
| ingest | 1024 | 12610 | 5 | 12,610 | 22 | 277,420 |
| predict | 1024 | 5392 | 5 | 5,392 | 22 | 118,624 |
| accuracy-job | 1024 | 49621 | 5 | 49,621 | 22 | 1091,662 |
| forecasts-api | 256 | 1299 | 71 | 1,299 | 1.800 | 584,550 |
| accuracy-api | 256 | 314 | 48 | 0,314 | 600 | 47,100 |
| chat-api | 512 | 4596 | 6 | 4,652 | 300 | 697,800 |
| **Tổng** | | | | | **2.766** | **2817,156** |

Dòng chat-api dùng thời gian tính tiền (billed duration); các hàm khác dùng thời gian chạy trung bình làm số đại diện. Dự toán chưa cộng thêm retry, khác biệt do cold start hay concurrency bất thường. Chỉ có hai lịch Scheduler vì predict được gọi qua Lambda Destination, không có lịch riêng.

##### 1.3. Lưu trữ, request và băng thông

| Hạng mục | Căn cứ | Giả định mỗi tháng |
|---|---|---|
| S3 storage | Khoảng 89,25 MB data lake + 1,06 MB asset của CDK | 0,10 GB-tháng |
| S3 Tier 1 / Tier 2 | 218 / 3381 request trong 17–23/11, nhân 30/7 | 934 / 14490 request |
| DynamoDB on-demand | 4 bảng; theo đầu vào đã lưu trong Calculator | 6381 RRU; 11310 WRU; 0,00158 GB-tháng |
| Amplify storage | 0,0039 GB trong 20 ngày app hoạt động, nhân 30/20 | 0,006 GB |
| Amplify outbound | 5.547.694 byte trong 2 ngày (22–23/11), quy về 30 ngày | 0,084 GB |
| ECR storage | 0,748 GB-tháng trong 23 ngày, nhân 30/23 | 0,98 GB-tháng |
| ECR outbound | 0,129 GB trong 23 ngày, nhân 30/23 | 0,168 GB |
| CloudWatch alarm | 6 alarm đang có | 6 standard alarm |
| CloudWatch custom metric | 2 metric filter đã cấu hình, chưa chắc có phát sinh phí | 2 metric (dự phòng) |
| CloudWatch GetMetricData / API / truy vấn log | 8 metric / 56 request / 0,0001 GB quét trong 23 ngày, nhân 30/23 | 11 metric / 74 request / 0,00014 GB |
| CloudWatch log | Lượng log lưu và vended log trong 23 ngày, nhân 30/23 | 0,00015 GB-tháng; 0,00028 GB |
| SNS | 9 request trong 23 ngày; 1 subscription email đã xác nhận | 12 request / 12 email (giả định) |
| KMS | 107 request trong 23 ngày; không có customer-managed key | 140 request (giả định) |
| SSM | 2 tham số Standard | 2 tham số, 0 USD |

**Băng thông:** hai dòng dữ liệu truyền ra được tính là Amplify 0,084 GB và ECR 0,168 GB, tổng 0,252 GB/tháng (đây là lượng dữ liệu, không phải tốc độ). S3 không ghi nhận dữ liệu vào/ra trong 17–23/11. API Gateway và SNS chưa có số đo egress riêng, nên dự toán không khẳng định chúng bằng 0 tuyệt đối. Amplify chỉ host web tĩnh, không có phút build tính tiền, nên không tính phí build, SSR hay WAF.

##### 1.4. Amazon Bedrock: tính theo token

Theo mã nguồn, router mặc định đưa mọi câu sang nhánh `complex`, dùng Claude Haiku 4.5 qua inference profile global, tối đa 3 vòng tool và 1500 token ra mỗi lần gọi. Nova 2 Lite có trong code nhưng chưa có usage, nên không có dòng riêng. Dự toán không giả định prompt caching, đầu vào ảnh hay Guardrails.

| Thông số | Giá trị |
|---|---:|
| CloudWatch 17–23/11 | 15 lần gọi model; 82.198 token vào / 3.708 token ra |
| Mẫu 22–23/11 (đối chiếu với log chat) | 7 câu chat thành công; 13 lần gọi; 71.904 token vào / 3.191 token ra |
| Số lần gọi model TB mỗi câu | 13/7 = 1,857 |
| Token vào TB mỗi câu | 10.272 |
| Token ra TB mỗi câu | 455,857 |
| 300 câu/tháng | khoảng 557 lần gọi; 3.081.600 token vào / khoảng 136.757 token ra |
| Giá trong Calculator (Global On-Demand) | 1 USD/1 triệu token vào; 5 USD/1 triệu token ra |
| **Chi phí** | **3,0816 + 0,684 = 3,7656 USD/tháng** |

Trong Pricing Calculator public, tổng token được nhập dưới dạng 1.800 request tương đương (1 request/phút, 1 giờ/ngày, 1712 token vào và 76 token ra mỗi request), không phải số lần gọi thực tế. Phần token ra lệch nhẹ vì Calculator chỉ nhận số nguyên. Mẫu 7 câu còn nhỏ và chưa đại diện mọi tình huống; câu hỏi dài hoặc nhiều vòng tool sẽ làm chi phí tăng.

##### 1.5. EventBridge Scheduler

Hai lịch `cpaws-dev-ingest-daily` và `cpaws-dev-accuracy-daily` đều Enabled, chạy `TUE-SAT`. Lịch accuracy mới tạo ngày 11/11, nên dự toán không suy ra từ số lượt đã chạy mà dùng 44 lượt/tháng theo cron hiện tại. Pricing Calculator public ghi 44 lượt/tháng, payload 1 KB, nằm trong Free Tier 14 triệu lượt, nên chi phí là 0 USD. Retry ngoài mức bình thường chưa được tính.

#### 2. Đủ 34 dòng dự toán

Số lượng và chi phí lấy nguyên từ bảng xuất của Pricing Calculator, không làm tròn giữa chừng; nhóm Lambda được viết gọn theo tên hàm.

| Service code | Nhóm / hàm | Usage type | Operation | Số lượng/tháng | USD/tháng |
| --- | --- | --- | --- | --- | --- |
| AWSLambda | ingest | APS1-Lambda-GB-Second | — | 277.42 Lambda-GB-Second | 0.004623676 |
| AWSLambda | ingest | APS1-Request | — | 22 Request | 0.0000044 |
| AWSLambda | accuracy-api | APS1-Lambda-GB-Second | — | 47.1 Lambda-GB-Second | 0.000785002 |
| AWSLambda | accuracy-api | APS1-Request | — | 600 Request | 0.00012 |
| AWSLambda | accuracy-job | APS1-Lambda-GB-Second | — | 1091.662 Lambda-GB-Second | 0.018194403 |
| AWSLambda | accuracy-job | APS1-Request | — | 22 Request | 0.0000044 |
| AWSLambda | chat-api | APS1-Lambda-GB-Second | — | 697.8 Lambda-GB-Second | 0.011630023 |
| AWSLambda | chat-api | APS1-Request | — | 300 Request | 0.00006 |
| AWSLambda | forecasts-api | APS1-Lambda-GB-Second | — | 584.55 Lambda-GB-Second | 0.009742519 |
| AWSLambda | forecasts-api | APS1-Request | — | 1800 Request | 0.00036 |
| AWSLambda | predict | APS1-Lambda-GB-Second | — | 118.624 Lambda-GB-Second | 0.001977071 |
| AWSLambda | predict | APS1-Request | — | 22 Request | 0.0000044 |
| AmazonApiGateway | CPAWS-API-Gateway | APS1-ApiGatewayRequest | ApiGatewayRequest | 2700 APS1-ApiGatewayRequest | 0.011475 |
| AmazonDynamoDB | CPAWS-DynamoDB | APS1-ReadRequestUnits | PayPerRequestThroughput | 6381 ReadRequestUnits | 0.000909292 |
| AmazonDynamoDB | CPAWS-DynamoDB | APS1-TimedStorage-ByteHrs | — | 0.00158 GB-Mo | 0 |
| AmazonDynamoDB | CPAWS-DynamoDB | APS1-WriteRequestUnits | PayPerRequestThroughput | 11310 WriteRequestUnits | 0.0080301 |
| AmazonS3 | CPAWS-S3 | APS1-Requests-Tier1 | — | 934 Requests | 0.00467 |
| AmazonS3 | CPAWS-S3 | APS1-Requests-Tier2 | — | 14490 Requests | 0.005796 |
| AmazonS3 | CPAWS-S3 | APS1-TimedStorage-ByteHrs | — | 0.1 GB-Mo | 0.0025 |
| AWSAmplify | CPAWS-Amplify | APS1-DataStorage | — | 0.006 GigaBytes | 0.000138 |
| AWSAmplify | CPAWS-Amplify | APS1-DataTransferOut | — | 0.084 GigaBytes | 0.0126 |
| AmazonCognito | CPAWS-Cognito | APS1-CognitoEssentialsMAU | CognitoEssentialsOperation | 5 CognitoUserPoolsMAU | 0.075 |
| AmazonECR | CPAWS-ECR | APS1-TimedStorage-ByteHrs | — | 0.98 GB-Mo | 0.098 |
| AWSDataTransfer | CPAWS-ECR | APS1-DataTransfer-Out-Bytes | — | 0.168 GB | 0.02016 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-CW:AlarmMonitorUsage | — | 6 Alarms | 0.6 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-CW:GMD-Metrics | GetMetricData | 11 Metrics | 0.00011 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-CW:MetricMonitorUsage | — | 2 Metrics | 0.6 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-CW:Requests | — | 74 Requests | 0.00074 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-DataScanned-Bytes | StartQuery | 0.00014 GB | 0.00000098 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-TimedStorage-ByteHrs | — | 0.00015 GB-Mo | 0.0000045 |
| AmazonCloudWatch | CPAWS-CloudWatch | APS1-VendedLog-Bytes | PutLogEvents | 0.00028 GB | 0.000196 |
| AmazonSNS | CPAWS-SNS | APS1-DeliveryAttempts-SMTP | — | 12 Notifications | 0 |
| AmazonSNS | CPAWS-SNS | APS1-Requests-Tier1 | — | 12 Requests | 0 |
| awskms | CPAWS-KMS | ap-southeast-1-KMS-Requests | — | 140 Requests | 0.00042 |
| **Tổng phần Console** | | | | 34 dòng | 1.488255766 |

*Bảng giữ nguyên định dạng số của file xuất (dấu chấm thập phân).*

#### Tài liệu tham khảo

- [AWS Pricing Calculator](https://calculator.aws/) và [AWS Well-Architected – Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)
- [Amazon Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/)
- [Amazon EventBridge Pricing](https://aws.amazon.com/eventbridge/pricing/)
- [AWS Systems Manager Pricing](https://aws.amazon.com/systems-manager/pricing/) và [AWS KMS Pricing](https://aws.amazon.com/kms/pricing/)
- Bảng giá của [Lambda](https://aws.amazon.com/lambda/pricing/), [API Gateway](https://aws.amazon.com/api-gateway/pricing/), [DynamoDB](https://aws.amazon.com/dynamodb/pricing/), [S3](https://aws.amazon.com/s3/pricing/), [Amplify](https://aws.amazon.com/amplify/pricing/), [Cognito](https://aws.amazon.com/cognito/pricing/), [ECR](https://aws.amazon.com/ecr/pricing/), [CloudWatch](https://aws.amazon.com/cloudwatch/pricing/), [SNS](https://aws.amazon.com/sns/pricing/)
- [AWS Budgets](https://aws.amazon.com/aws-cost-management/aws-budgets/) và [AWS Cost Anomaly Detection](https://aws.amazon.com/aws-cost-management/aws-cost-anomaly-detection/)

*Mốc giá 23–24/11/2026; bảng giá và ưu đãi có thể thay đổi sau đó.*
