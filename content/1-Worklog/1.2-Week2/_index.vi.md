---
title: "Worklog Tuần 2"
date: 2026-09-21
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### MỤC TIÊU TUẦN 2:
- Thực hành quản lý truy cập với AWS IAM (user, group, role).
- Dựng VPC với đầy đủ subnet, route table, gateway và các lớp bảo mật.
- Kết nối AWS với mạng on-premises giả lập qua AWS Site-to-Site VPN.
- Cùng nhóm lên ý tưởng cho dự án.

### CÁC CÔNG VIỆC TRONG TUẦN 2:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:---:|---|:---:|:---:|---|
| **2** | **- Thực hành lab: Quản lý truy cập với AWS IAM**<br>+ IAM group, user, policy và role<br>+ Tạo Admin group và Admin user, đăng nhập bằng Admin user<br>+ Tạo Admin role và OperatorUser<br>+ Chuyển đổi role: cho phép OperatorUser switch role, đăng nhập và chuyển role<br>+ Dọn dẹp tài nguyên | 21/09/2026 | 21/09/2026 | [Quản trị quyền truy cập với IAM](https://000002.awsfcaj.com/vi/) |
| **3** | **- Thực hành lab: Amazon VPC và AWS Site-to-Site VPN (phần 1)**<br>+ Các thành phần của Amazon VPC: subnet, route table, Internet Gateway, NAT Gateway<br>+ Bảo mật mạng: security group, network ACL, VPC Resource Map<br>+ Chuẩn bị môi trường:<br>* Tạo VPC, subnet, Internet Gateway và route table<br>* Tạo security group<br>* Bật VPC Flow Logs | 22/09/2026 | 22/09/2026 | [Bắt đầu với Amazon Virtual Private Cloud (VPC) và AWS Site-to-Site VPN](https://000003.awsfcaj.com/vi/) |
| **4** | **- Thực hành lab: Amazon VPC và AWS Site-to-Site VPN (phần 2)**<br>+ Triển khai Amazon EC2 trong VPC:<br>* Tạo EC2 instance và kiểm tra kết nối<br>* Tạo NAT Gateway Multi-AZ<br>* Sử dụng Reachability Analyzer<br>* Cấu hình EC2 Instance Connect Endpoint<br>* Kết nối bằng AWS Systems Manager Session Manager<br>* Thiết lập giám sát và cảnh báo với CloudWatch | 23/09/2026 | 23/09/2026 | [Bắt đầu với Amazon Virtual Private Cloud (VPC) và AWS Site-to-Site VPN](https://000003.awsfcaj.com/vi/) |
| **5** | **- Thực hành lab: Amazon VPC và AWS Site-to-Site VPN (phần 3)**<br>+ Tạo môi trường VPN: một VPC riêng và EC2 đóng vai trò phía khách hàng<br>+ Cấu hình kết nối VPN:<br>* Tạo Virtual Private Gateway và Customer Gateway<br>* Tạo VPN connection<br>* Cấu hình Customer Gateway<br>* Tùy chỉnh VPN tunnel và các tùy chọn VPN nâng cao | 24/09/2026 | 24/09/2026 | [Bắt đầu với Amazon Virtual Private Cloud (VPC) và AWS Site-to-Site VPN](https://000003.awsfcaj.com/vi/) |
| **6** | **- Thực hành lab: Amazon VPC và AWS Site-to-Site VPN (phần 4)**<br>+ Xử lý sự cố VPN theo hướng dẫn của lab và tài liệu chính thức của AWS<br>+ Cấu hình VPN bằng strongSwan với Transit Gateway (phần tùy chọn):<br>* Tạo Customer Gateway và Transit Gateway<br>* Tạo VPN connection và Transit Gateway attachment<br>* Cấu hình route table và Customer Gateway<br>+ Xem các template Infrastructure as Code đi kèm lab<br>+ Dọn dẹp tài nguyên<br>**- Họp nhóm: lên ý tưởng cho dự án**<br>+ Mỗi thành viên đề xuất ý tưởng, cả nhóm thảo luận dữ liệu và dịch vụ AWS mà mỗi ý tưởng cần | 25/09/2026 | 25/09/2026 | [Bắt đầu với Amazon Virtual Private Cloud (VPC) và AWS Site-to-Site VPN](https://000003.awsfcaj.com/vi/) |

### KẾT QUẢ ĐẠT ĐƯỢC TRONG TUẦN 2:
1. Tạo IAM user, group, role và thực hành chuyển đổi role.
2. Dựng VPC với subnet, route table, Internet Gateway, NAT Gateway và bảo vệ bằng security group, network ACL.
3. Chạy EC2 trong VPC và kết nối qua Session Manager, EC2 Instance Connect Endpoint.
4. Thiết lập Site-to-Site VPN giữa AWS và mạng on-premises giả lập, thử thêm cách dùng strongSwan với Transit Gateway.
5. Chia sẻ ý tưởng dự án với nhóm. Thứ 7 ngày 26/09 tôi tham gia thêm buổi Buildathon kickoff (Sự kiện 1).
