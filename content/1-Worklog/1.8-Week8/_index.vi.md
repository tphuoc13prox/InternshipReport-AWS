---
title: "Worklog Tuần 8"
date: 2026-11-02
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:
- Tự động hóa triển khai serverless với AWS CodePipeline.
- Học Infrastructure as Code với CloudFormation và AWS CDK, triển khai kiến trúc 3 tầng.
- Họp team project về cách hiển thị, thiết kế dữ liệu và pipeline thu thập giá.
- Chuẩn bị demo chatbot ngày 08/11 (kịch bản, slide) và viết chương chatbot trong báo cáo bảo vệ.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 | **- Thực hành lab: CI/CD cho Serverless với AWS CodePipeline**<br>+ Chuẩn bị<br>+ Xây dựng SAM pipeline: tạo Git repository và SAM pipeline<br>+ Kiểm tra và dọn dẹp tài nguyên<br>**- Họp team project: Định hướng cách hiển thị và thiết kế dữ liệu**<br>+ Web hiển thị q10/q50/q90 quy đổi thành khoảng giá, xác suất tăng, chuyển USD/VND<br>+ Thiết kế bảng DynamoDB `Forecasts` (PK `asset`, SK `base_date`)<br>+ Chốt nguồn giá: GLD, SLV, BZ=F, CL=F, VND=X; đối chiếu với EIA<br>**- Dự án nhóm: kịch bản demo chatbot ngày 08/11**<br>+ Kịch bản demo cố định cho chatbot<br>+ Kết quả dev của prompt Haiku v3 và kiểm tra nhóm câu hỏi loại A với prompt v2<br>+ Gán nhãn lại hai tin trong bộ tin demo theo kế hoạch labeling-v3 | 02/11/2026 | 02/11/2026 | [Serverless - CI/CD với CodePipeline](https://000084.awsfcaj.com/vi/)<br>[Claude parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-claude.html) |
| 3 | **- Thực hành lab: AWS CloudFormation**<br>+ Chuẩn bị: tạo IAM user<br>+ CloudFormation cơ bản: khởi chạy AWS CloudShell, tạo CloudFormation template<br>+ CloudFormation nâng cao:<br>* Custom resource với Lambda function, tạo stack, kết nối EC2 instance<br>* Mappings và StackSets<br>* Drift detection<br>+ Dọn dẹp tài nguyên<br>**- Review nhóm: kịch bản demo và khung chat**<br>+ Sửa kịch bản demo theo review (PR #77)<br>+ Nhóm đã kết nối ChatPanel trên web với POST /chat | 03/11/2026 | 03/11/2026 | [AWS CloudFormation](https://000037.awsfcaj.com/vi/)<br>[AWS Amplify Hosting](https://docs.aws.amazon.com/amplify/latest/userguide/welcome.html) |
| 4 | **- Thực hành lab: AWS CDK cơ bản**<br>+ CDK cơ bản: tạo workspace, cấu hình AWS CloudShell<br>+ Tạo CDK template<br>+ Cập nhật CDK template<br>+ Dọn dẹp tài nguyên<br>**- Dự án nhóm: slide demo và luật cho router**<br>+ Slide Marp cho buổi demo chatbot<br>+ Khóa bộ luật của router trước lần chạy held-out | 04/11/2026 | 04/11/2026 | [CDK cơ bản](https://000038.awsfcaj.com/vi/) |
| 5 | **- Họp team project: Xây dựng pipeline thu thập giá theo thiết kế dữ liệu**<br>+ CLI `ingest_prices` ghi dữ liệu vào S3 (`raw/`, `clean/prices/`), dùng lịch giao dịch NYSE/CME<br>+ Bỏ thanh của ngày hiện tại theo giờ New York, gắn cờ dòng bất thường, không forward-fill target<br>+ Đóng gói Lambda ingest dạng container image, ghim digest base image<br>**- Dự án nhóm: chương chatbot cho báo cáo bảo vệ**<br>+ Viết chương chatbot và sửa theo review<br>+ Thêm báo cáo tuần mới nhất của EIA vào bộ tin demo | 05/11/2026 | 05/11/2026 | [Lambda container image](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html)<br>[Generative AI Lens](https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/generative-ai-lens.html) |
| 6 | **- Thực hành lab: Giới thiệu Infrastructure as Code**<br>+ Chuẩn bị<br>+ Tạo Lambda function<br>+ Tạo VPC và EC2<br>+ Triển khai kiến trúc 3 tầng bằng CloudFormation stack<br>+ Kiểm tra truy cập tầng web, tầng ứng dụng và tầng database<br>+ Dọn dẹp tài nguyên<br>**- Dự án nhóm: kết quả held-out của router và prompt Haiku v4**<br>+ Ghi buổi 08/11 là buổi chạy thử chatbot nội bộ trong tài liệu bộ tin demo<br>+ Ghi lại kết quả held-out của router (chưa đạt)<br>+ Prompt Haiku v4: văn bản thuần, mẫu câu trả lời, ngày mặc định cho get_news (hoàn thành trong cuối tuần) | 06/11/2026 | 06/11/2026 | [Giới thiệu Infrastructure as Code với AWS CloudFormation](https://000102.awsfcaj.com/vi/)<br>[Bedrock prompt engineering](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html) |

### Kết quả đạt được tuần 8:
1. Dựng SAM pipeline bằng CodePipeline.
2. Viết CloudFormation template có custom resource, mappings, StackSets và drift detection.
3. Định nghĩa hạ tầng bằng code với AWS CDK và triển khai kiến trúc 3 tầng bằng CloudFormation.
4. Chuẩn bị kịch bản và slide demo chatbot, viết chương chatbot trong báo cáo bảo vệ và chuyển sang prompt Haiku v4.
5. Cùng nhóm chốt cách hiển thị dự báo, bảng DynamoDB `Forecasts`, nguồn giá và pipeline thu thập giá.
