---
title: "Worklog Tuần 5"
date: 2026-10-12
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:
- Host website tĩnh trên S3 và CloudFront.
- Học các dịch vụ bảo mật của AWS và kiểm soát chi phí bằng IAM policy.
- Học các dịch vụ cơ sở dữ liệu của AWS và thực hành Amazon RDS.
- Thêm CI cho repository dự án và review các pull request đầu tiên về hạ tầng và API.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 | **- Thực hành lab: Host website tĩnh với Amazon S3**<br>+ Tạo S3 bucket và tải lên các file của website<br>+ Bật static website hosting, cấu hình Block Public Access và object công khai, kiểm tra website<br>+ Tăng tốc website với Amazon CloudFront: chặn truy cập công khai vào S3, cấu hình và kiểm tra CloudFront<br>+ Bucket versioning, di chuyển object, sao chép liên Region (Cross-Region Replication)<br>+ Dọn dẹp tài nguyên, ghi chú và best practice<br>**- Dự án nhóm: thêm CI cho repository**<br>+ Thêm workflow GitHub Actions chạy pytest và ruff cho mỗi push và pull request<br>+ Review và merge PR #1 (CEEMDAN cache và Lambda ingest) | 12/10/2026 | 12/10/2026 | [Khởi Đầu Với Amazon S3](https://000057.awsfcaj.com/vi/)<br>[Lambda container images](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html) |
| 3 | **- Học Module 05: Dịch vụ bảo mật trên AWS**<br>+ 05-01: Mô hình trách nhiệm chung (Shared Responsibility Model)<br>+ 05-02: AWS Identity and Access Management<br>+ 05-03: Amazon Cognito<br>+ 05-04: AWS Organizations<br>+ 05-05: AWS IAM Identity Center<br>+ 05-06: AWS Key Management Service<br>+ 05-07: AWS Security Hub | 13/10/2026 | 13/10/2026 | [Quản trị quyền truy cập với IAM](https://000002.awsfcaj.com/vi/)<br>[Module 05-01](https://youtu.be/tsobAlSg19g?si=-xSAVT8MZReV10RP) |
| 4 | **- Thực hành lab: Quản trị chi phí và tài nguyên bằng IAM**<br>+ Chuẩn bị: tạo IAM group và IAM user<br>+ Giới hạn theo Region: tạo policy, gắn vào group, kiểm tra hiệu lực<br>+ Giới hạn theo EC2 instance family<br>+ Giới hạn theo kích thước instance<br>+ Giới hạn theo loại EBS volume<br>+ Dọn dẹp tài nguyên | 14/10/2026 | 14/10/2026 | [Quản trị Chi phí và Sử dụng tài nguyên](https://000064.awsfcaj.com/vi/) |
| 5 | **- Học Module 06: Dịch vụ cơ sở dữ liệu trên AWS**<br>+ 06-01: Ôn lại các khái niệm cơ sở dữ liệu<br>+ 06-02: Amazon RDS và Amazon Aurora<br>+ 06-03: Amazon Redshift và Amazon ElastiCache<br>**- Họp nhóm: tài khoản AWS, quyền IAM và data lake**<br>+ Đọc ADR-0006 (tài khoản AWS, quyền IAM và credit)<br>+ Xem CDK stack CpawsData (S3 data lake và ECR repository cho image ingest) | 15/10/2026 | 15/10/2026 | [Bắt đầu với Amazon RDS](https://000005.awsfcaj.com/vi/)<br>[Module 06-01](https://youtu.be/OOD2RwWuLRw?si=gUgPzH3nVghQiDQC) |
| 6 | **- Thực hành lab: Cơ sở dữ liệu cơ bản với Amazon RDS**<br>+ Chuẩn bị: tạo VPC, security group cho EC2 và RDS, DB subnet group<br>+ Tạo EC2 instance và RDS database instance<br>+ Triển khai ứng dụng và kết nối tới RDS<br>+ Sao lưu và khôi phục (backup/restore)<br>+ Dọn dẹp tài nguyên<br>**- Review nhóm: merge các PR hạ tầng và API contract**<br>+ Review và merge PR #4 (seed CEEMDAN và đối chiếu chéo), PR #5 (hạ tầng CpawsData) và PR #6 (API contract) | 16/10/2026 | 16/10/2026 | [Bắt đầu với Amazon RDS](https://000005.awsfcaj.com/vi/)<br>[API Gateway REST API](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-rest-api.html) |

### Kết quả đạt được tuần 5:
1. Host website tĩnh trên S3, đặt CloudFront phía trước, thử versioning và sao chép liên Region.
2. Hoàn thành Module 05: Shared Responsibility Model, IAM, Cognito, Organizations, Identity Center, KMS, Security Hub.
3. Viết IAM policy giới hạn Region, instance family, kích thước instance và loại EBS để kiểm soát chi phí.
4. Hoàn thành Module 06 và triển khai cơ sở dữ liệu RDS có sao lưu, khôi phục.
5. Thêm workflow CI (pytest + ruff) cho repository của nhóm, review và merge các PR đầu tiên về hạ tầng và API contract.
