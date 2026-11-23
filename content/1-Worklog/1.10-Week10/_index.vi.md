---
title: "Worklog Tuần 10"
date: 2026-11-16
publishDate: 2026-11-13
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:
- Dùng AWS Toolkit và Amazon Q trong VS Code.
- Họp team project để chốt kết quả đánh giá và cách cho người chấm truy cập web.
- Phần chatbot: rà giới hạn lượt dùng, review tài liệu và cập nhật bộ tin demo.
- Bắt đầu viết Worklog cá nhân theo yêu cầu báo cáo của nhóm.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|:---:|---|:---:|:---:|---|
| 2 | **- Họp team project: Chốt kết quả đánh giá và mở tài khoản cho người chấm**<br>+ Duyệt kết quả test2020, giữ mô hình `baseline-q252`<br>+ Review các PR báo cáo và tính năng tự đăng ký trên web<br>+ Thống nhất mở tự đăng ký Cognito trong thời gian chấm (ADR-0014)<br>**- Dự án nhóm: rà giới hạn lượt dùng chatbot**<br>+ Trả lời Tuấn về giới hạn 20 lượt/người/ngày và 100 lượt/ngày của chatbot, dùng cho ADR-0014<br>+ Đối chiếu lại với code quota, cấu hình API stack và unit test | 16/11/2026 | 16/11/2026 | [DynamoDB items](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html) |
| 3 | **- Dự án nhóm: review tài liệu chatbot và README**<br>+ Review và approve PR của Khải chỉnh lại tài liệu chatbot<br>+ Review và approve PR của Khải viết lại README: tổng quan, kiến trúc, cách chạy local<br>+ Tự hỏi chatbot trên web để kiểm câu trả lời về tin FOMC đầu tháng 10 (đạt)<br>+ Nhận yêu cầu báo cáo từ Khải: Proposal và Workshop viết chung cho cả nhóm, Worklog cá nhân, song ngữ, có mục COST | 17/11/2026 | 17/11/2026 |  |
| 4 | **- Thực hành lab: AWS Toolkit for VS Code với Amazon Q**<br>+ Cài AWS Toolkit cho Visual Studio Code<br>+ Kết nối tài khoản AWS và đổi Region<br>+ Xác thực bằng IAM credentials và AWS Builder ID<br>+ Làm việc với dịch vụ AWS: AWS Explorer, Amazon Q, Amazon CodeWhisperer | 18/11/2026 | 18/11/2026 | [AWS Toolkit for VS Code: Amazon Q & CodeWhisperer](https://000087.awsfcaj.com/vi/) |
| 5 | **- Báo cáo: viết Worklog cá nhân theo yêu cầu báo cáo của nhóm**<br>+ Rà các tuần đã qua, ghi lại việc theo từng ngày bằng tiếng Anh và tiếng Việt | 19/11/2026 | 19/11/2026 | [FCAJ HCM Rules](https://hcm-rules.awsfcaj.com/) |
| 6 | **- Dự án nhóm: cập nhật bộ tin demo của chatbot**<br>+ Tra các tin mới của Fed và EIA sau đợt cập nhật 11/11, chọn 2 tin và gắn nhãn tay theo ADR-0013<br>+ Mở PR #142: thêm biên bản họp FOMC 27–28/10 (vàng, bạc) và báo cáo WPSR 18/11 (Brent, WTI) vào các file tin của Lambda chat, cập nhật tài liệu bộ tin demo, thêm 1 test<br>+ Khải approve, tôi merge PR | 20/11/2026 | 20/11/2026 |  |

### Kết quả đạt được tuần 10:
1. Kết nối VS Code với AWS qua AWS Toolkit (IAM credentials, AWS Builder ID) và dùng thử Amazon Q.
2. Cùng nhóm chốt kết quả đánh giá, giữ mô hình `baseline-q252` và thống nhất mở tự đăng ký Cognito trong thời gian chấm (ADR-0014).
3. Xác nhận giới hạn lượt dùng chatbot với code và unit test để đưa vào ADR-0014.
4. Review tài liệu chatbot và README của nhóm; cập nhật bộ tin demo của chatbot với biên bản FOMC 27–28/10 và báo cáo WPSR 18/11 (PR #142).
5. Bắt đầu viết Worklog cá nhân song ngữ theo yêu cầu báo cáo của nhóm.
