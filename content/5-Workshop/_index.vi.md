---
title: "Workshop"
date: 2026-11-20
weight: 5
chapter: false
pre: " <b> 5. </b> "
---
# CPAWS - Xây dựng nền tảng dự báo giá hàng hóa serverless trên AWS

#### Tổng quan

**CPAWS** là ứng dụng web dự báo giá vàng, bạc, dầu Brent và dầu WTI cho 1–7 phiên giao dịch tới, hiển thị theo USD và VND, kèm chatbot tiếng Việt trên Amazon Bedrock để giải thích dự báo. Hệ thống chạy serverless theo lịch: không có endpoint, máy chủ hay NAT Gateway chạy 24/7.

Workshop này mô tả đúng hệ thống mà nhóm CPAWS (5 thành viên, chương trình FCAJ) đang chạy trên tài khoản AWS của nhóm tại region `ap-southeast-1`. Mọi lệnh, cấu hình và số đo lấy từ mã nguồn và các lượt chạy thật; phần nào chưa làm hoặc làm chưa đạt thì ghi rõ.

Trong workshop này, bạn sẽ học cách:
- Thu thập giá theo lịch giao dịch, làm sạch và lưu vào data lake trên **Amazon S3**
- Xây mốc dự báo phân vị không rò rỉ dữ liệu tương lai, đánh giá bằng pinball loss
- Triển khai hạ tầng bằng **AWS CDK**: S3, ECR, **AWS Lambda** dạng container image, **Amazon DynamoDB**
- Tự động hóa luồng hằng ngày bằng **Amazon EventBridge Scheduler** và Lambda Destinations
- Mở API có xác thực bằng **Amazon API Gateway** và **Amazon Cognito**, đưa web lên **AWS Amplify Hosting**
- Gọi **Amazon Bedrock** Converse API với tool use, để chatbot chỉ dùng số liệu của hệ thống
- Kiểm thử, giám sát bằng **Amazon CloudWatch**, ước tính và theo dõi chi phí, đối chiếu với AWS Well-Architected

#### Nội dung

1. [Giới thiệu](5.1-workshop-overview/)
2. [Yêu cầu tiên quyết](5.2-prerequiste/)
3. [Dữ liệu và mô hình](5.3-data-and-model/)
4. [Triển khai hạ tầng với CDK](5.4-infrastructure/)
5. [Chatbot với Amazon Bedrock](5.5-chatbot/)
6. [Kiểm thử và giám sát](5.6-testing/)
7. [Chi phí và Well-Architected](5.7-cost/)
8. [Dọn dẹp tài nguyên](5.8-cleanup/)

#### Mã nguồn

Repo của nhóm: [CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting](https://github.com/CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting) (nhánh `developer`).
