---
title: "Worklog Tuần 4"
date: 2026-10-05
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:
- Tìm hiểu sâu Amazon EC2: loại instance, AMI, EBS, user data, Auto Scaling.
- Triển khai ứng dụng web trên EC2 Linux và Windows, quản lý tài nguyên bằng AWS CLI.
- Học các dịch vụ lưu trữ của AWS (Module 04).
- Tạo repository dự án và cùng nhóm hoàn thiện proposal.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 | **- Học Module 03: Dịch vụ máy ảo (Compute) trên AWS**<br>+ 03-01: Máy ảo (Compute VM) trên AWS<br>+ 03-01-01: Các loại EC2 instance<br>+ 03-01-02: AMI, sao lưu, key pair<br>+ 03-01-03: Elastic Block Store (EBS)<br>+ 03-01-04: EC2 Instance Store<br>+ 03-01-05: EC2 user data<br>+ 03-01-06: EC2 metadata<br>+ 03-01-07: EC2 Auto Scaling<br>+ 03-02: EC2 Auto Scaling, EFS/FSx, Lightsail, MGN<br>**- Dự án nhóm: tạo repository CPAWS-commodity-forecasting**<br>+ Tạo repository GitHub cho nhóm (initial commit) | 05/10/2026 | 05/10/2026 | [Giới thiệu về Amazon EC2](https://000004.awsfcaj.com/vi/)<br>[Module 03-01](https://youtu.be/-t5h4N6vfBs?si=bS-jetOxckl24fDr) |
| 3 | **- Thực hành lab: Amazon EC2 (phần 1)**<br>+ Chuẩn bị: tạo VPC và security group cho Linux và Windows<br>+ Khởi tạo và kết nối tới Windows instance và Linux instance<br>+ Các thao tác cơ bản với Amazon EC2:<br>* Thay đổi cấu hình instance (resize)<br>* Tạo EBS snapshot và custom AMI, khởi tạo instance từ custom AMI<br>* Truy cập lại khi mất key pair (Windows qua SSM, Linux qua user data)<br>* Cài giao diện desktop cho EC2 Ubuntu 22.04<br>* Lưu trữ (archive) EBS snapshot và chia sẻ AMI | 06/10/2026 | 06/10/2026 | [Giới thiệu về Amazon EC2](https://000004.awsfcaj.com/vi/) |
| 4 | **- Thực hành lab: Amazon EC2 (phần 2)**<br>+ Triển khai ứng dụng Node.js trên Amazon Linux: cài LAMP stack và phpMyAdmin, cài Node.js, triển khai ứng dụng<br>+ Triển khai ứng dụng Node.js trên EC2 Windows: cài XAMPP và Node.js, triển khai ứng dụng<br>+ Quản trị việc sử dụng tài nguyên bằng IAM: giới hạn theo Region, instance family, instance type, loại EBS volume, địa chỉ IP công ty và khung thời gian<br>+ Dọn dẹp tài nguyên<br>**- Review nhóm: cấu trúc repository và proposal bản cuối**<br>+ Review cấu trúc thư mục (proposal, architecture, ADR, defense), cấu hình Python và kiểm tra trước khi commit<br>+ Proposal bản cuối được đưa lên repository | 07/10/2026 | 07/10/2026 | [Giới thiệu về Amazon EC2](https://000004.awsfcaj.com/vi/)<br>[Proposal Sample](https://workshop-sample.awsfcaj.com/2-proposal/) |
| 5 | **- Thực hành lab: Làm quen với AWS CLI**<br>+ Cài đặt và cấu hình AWS CLI<br>+ Kiểm tra tài nguyên bằng CLI<br>+ Dùng AWS CLI với Amazon S3, Amazon SNS và IAM<br>+ Dùng AWS CLI với VPC và Internet Gateway<br>+ Tạo EC2 instance bằng AWS CLI<br>+ Xử lý lỗi và dọn dẹp tài nguyên | 08/10/2026 | 08/10/2026 | [Làm quen với AWS CLI](https://000011.awsfcaj.com/vi/) |
| 6 | **- Học Module 04: Dịch vụ lưu trữ trên AWS**<br>+ 04-02: Amazon S3, access point, storage class<br>+ 04-03: S3 static website và CORS, kiểm soát truy cập, object key và hiệu năng, Glacier<br>+ 04-04: Snow Family, Storage Gateway, AWS Backup<br>**- Review nhóm: pipeline thu thập giá**<br>+ Review pipeline thu thập giá (yfinance cho GLD, SLV, CL=F, BZ=F, VND=X, đối chiếu chéo với EIA) và phần EDA giá<br>+ Đọc ADR-0005 về nguồn dữ liệu giá | 09/10/2026 | 09/10/2026 | [Khởi Đầu Với Amazon S3](https://000057.awsfcaj.com/vi/)<br>[Module 04-02](https://youtu.be/_yunukwcAwc?si=6HqDHVQJq8Ks9aWh) |

### Kết quả đạt được tuần 4:
1. Nắm các thành phần của EC2 (loại instance, AMI, EBS, instance store, user data, metadata, Auto Scaling).
2. Thực hành resize, snapshot, custom AMI và truy cập lại instance khi mất key pair.
3. Triển khai ứng dụng Node.js trên EC2 Linux (LAMP) và Windows (XAMPP), giới hạn việc dùng tài nguyên bằng điều kiện IAM.
4. Tạo tài nguyên VPC, S3, SNS, IAM và EC2 bằng AWS CLI.
5. Hoàn thành Module 04: S3, storage class, Glacier, Snow Family, Storage Gateway, AWS Backup.
6. Tạo repository cho nhóm; trong tuần nhóm đã đưa proposal bản cuối và pipeline thu thập dữ liệu đầu tiên lên repo.
