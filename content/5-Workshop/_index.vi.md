---
title: "Workshop"
date: 2025-09-09
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# CPAWS – Dự báo giá hàng hóa trên AWS

#### Tổng quan

Workshop này đi qua **CPAWS**, hệ thống serverless mà nhóm chúng tôi xây dựng và đang chạy trên AWS. Mỗi ngày giao dịch, hệ thống thu thập giá vàng, bạc, dầu Brent và WTI, phát hành khoảng dự báo 1–7 phiên cho từng tài sản, đo xem các dự báo đã phát hành đáng tin đến đâu, rồi đưa tất cả lên một web app có chatbot tiếng Việt chạy trên Amazon Bedrock.

Bạn sẽ dựng lại đúng các khối mà nhóm dùng, theo đúng thứ tự: lưu trữ và bảng dữ liệu, pipeline ingest → predict hằng ngày, job đo độ tin cậy, đăng nhập và REST API, chatbot, web trên AWS Amplify, cuối cùng là alarm, kiểm thử, chi phí và dọn dẹp. Toàn bộ hạ tầng viết bằng AWS CDK (Python), chia thành 5 stack: `CpawsServing`, `CpawsAuth`, `CpawsData`, `CpawsApi` và `CpawsOps`.

+ **Pipeline dữ liệu:** Amazon EventBridge Scheduler, AWS Lambda (image container trên Amazon ECR), Amazon S3, Amazon DynamoDB.
+ **Dịch vụ cho người dùng:** Amazon Cognito, Amazon API Gateway, AWS Lambda, Amazon Bedrock, AWS Amplify Hosting.
+ **Vận hành:** Amazon CloudWatch, Amazon SNS, AWS Systems Manager Parameter Store, AWS Budgets.

{{% notice note %}}
CPAWS là dự án nhóm 5 người. Bài viết mô tả toàn bộ hệ thống; phần nào do một thành viên cụ thể làm thì trang đó ghi rõ. Dự báo chỉ để tham khảo, không phải lời khuyên đầu tư.
{{% /notice %}}

#### Nội dung

1. [Tổng quan workshop](5.1-workshop-overview/)
2. [Chuẩn bị](5.2-prerequiste/)
3. [Kiến trúc và luồng dữ liệu](5.3-architecture/)
4. [Pipeline dữ liệu](5.4-data-pipeline/)
5. [API, đăng nhập và chatbot](5.5-api-and-chatbot/)
6. [Web trên AWS Amplify](5.6-web-amplify/)
7. [Giám sát và kiểm thử](5.7-monitoring-and-testing/)
8. [Dự toán và tối ưu chi phí](5.8-cost/)
9. [Demo](5.9-demo/)
10. [Dọn dẹp tài nguyên](5.10-cleanup/)
11. [Khó khăn và bài học](5.11-lessons-learned/)
