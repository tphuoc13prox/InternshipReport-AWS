---
title: "Worklog Tuần 2"
date: 2026-09-21
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:
- Thực hành quản lý truy cập với AWS IAM (user, group, role).
- Dựng VPC với đầy đủ subnet, route table, gateway và các lớp bảo mật.
- Kết nối AWS với mạng on-premises giả lập qua AWS Site-to-Site VPN.
- Lập team CPAWS, họp team project để lên ý tưởng và chọn hướng cho dự án.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 | **- Lên văn phòng AWS lần đầu tiên**<br>**- Thực hành lab: Quản lý truy cập với AWS IAM**<br>+ IAM group, user, policy và role<br>+ Tạo Admin group và Admin user, đăng nhập bằng Admin user<br>+ Tạo Admin role và OperatorUser<br>+ Chuyển đổi role: cho phép OperatorUser switch role, đăng nhập và chuyển role<br>+ Dọn dẹp tài nguyên | 21/09/2026 | 21/09/2026 | [Quản trị quyền truy cập với IAM](https://000002.awsfcaj.com/vi/) |
| 3 | **- Gặp gỡ và lập team**<br>+ Gặp các bạn cùng nhóm, lập team CPAWS gồm 5 thành viên<br>**- Thực hành lab: Amazon VPC và AWS Site-to-Site VPN (phần 1)**<br>+ Các thành phần của Amazon VPC: subnet, route table, Internet Gateway, NAT Gateway<br>+ Bảo mật mạng: security group, network ACL, VPC Resource Map<br>+ Chuẩn bị môi trường:<br>* Tạo VPC, subnet, Internet Gateway và route table<br>* Tạo security group<br>* Bật VPC Flow Logs | 22/09/2026 | 22/09/2026 | [Bắt đầu với Amazon Virtual Private Cloud (VPC) và AWS Site-to-Site VPN](https://000003.awsfcaj.com/vi/) |
| 4 | **- Thực hành lab: Amazon VPC và AWS Site-to-Site VPN (phần 2)**<br>+ Triển khai Amazon EC2 trong VPC:<br>* Tạo EC2 instance và kiểm tra kết nối<br>* Tạo NAT Gateway Multi-AZ<br>* Sử dụng Reachability Analyzer<br>* Cấu hình EC2 Instance Connect Endpoint<br>* Kết nối bằng AWS Systems Manager Session Manager<br>* Thiết lập giám sát và cảnh báo với CloudWatch | 23/09/2026 | 23/09/2026 | [Bắt đầu với Amazon Virtual Private Cloud (VPC) và AWS Site-to-Site VPN](https://000003.awsfcaj.com/vi/) |
| 5 | **- Thực hành lab: Amazon VPC và AWS Site-to-Site VPN (phần 3)**<br>+ Tạo môi trường VPN: một VPC riêng và EC2 đóng vai trò phía khách hàng<br>+ Cấu hình kết nối VPN:<br>* Tạo Virtual Private Gateway và Customer Gateway<br>* Tạo VPN connection<br>* Cấu hình Customer Gateway<br>* Tùy chỉnh VPN tunnel và các tùy chọn VPN nâng cao | 24/09/2026 | 24/09/2026 | [Bắt đầu với Amazon Virtual Private Cloud (VPC) và AWS Site-to-Site VPN](https://000003.awsfcaj.com/vi/) |
| 6 | **- Thực hành lab: Amazon VPC và AWS Site-to-Site VPN (phần 4)**<br>+ Xử lý sự cố VPN theo hướng dẫn của lab và tài liệu chính thức của AWS<br>+ Dọn dẹp tài nguyên<br>**- Họp team project: lên ý tưởng, phân tích, nghiên cứu để chọn lọc idea phù hợp**<br>+ Nhóm CPAWS (5 thành viên) thống nhất cách làm việc và kênh trao đổi<br>+ Thảo luận hướng dự báo giá hàng hóa giao dịch chung giữa thế giới và Việt Nam (vàng, bạc, dầu thô) do Khải đề xuất | 25/09/2026 | 25/09/2026 | [Bắt đầu với Amazon Virtual Private Cloud (VPC) và AWS Site-to-Site VPN](https://000003.awsfcaj.com/vi/)<br>[World Bank Commodity Markets](https://www.worldbank.org/en/research/commodity-markets) |

### Kết quả đạt được tuần 2:
1. Lên văn phòng AWS lần đầu tiên; tạo IAM user, group, role và thực hành chuyển đổi role.
2. Dựng VPC với subnet, route table, Internet Gateway, NAT Gateway và bảo vệ bằng security group, network ACL.
3. Chạy EC2 trong VPC và kết nối qua Session Manager, EC2 Instance Connect Endpoint.
4. Thiết lập Site-to-Site VPN giữa AWS và mạng on-premises giả lập, xử lý sự cố VPN theo hướng dẫn của AWS.
5. Lập team CPAWS (22/09) và họp team project lần đầu: thống nhất cách làm việc và thảo luận hướng dự báo giá hàng hóa. Thứ 7 ngày 26/09 em tham gia thêm buổi Buildathon kickoff (Sự kiện 1).
