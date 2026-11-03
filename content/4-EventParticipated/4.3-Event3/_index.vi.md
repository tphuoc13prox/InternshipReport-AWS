---
title: "Sự Kiện 3"
date: 2026-10-02
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Báo Cáo Tóm Tắt: "Fireside Chat with Dr. Werner: Navigating the Future of Cloud & AI in Vietnam"

### Mục tiêu sự kiện

- Hiểu cách các hệ thống công nghệ quy mô lớn được thiết kế và vận hành để đảm bảo độ tin cậy.
- Học các nguyên tắc thực tiễn về operational excellence từ kinh nghiệm kỹ thuật của Amazon.
- Hiểu cách AI đang thay đổi vai trò của kỹ sư phần mềm.
- Tìm hiểu những kỹ năng mà kỹ sư cần có để làm việc hiệu quả trong môi trường công nghệ do AI dẫn dắt.
- Hiểu tầm quan trọng của tư duy hệ thống, học hỏi liên tục, tinh thần làm chủ (ownership) và giao tiếp.

### Diễn giả

- **Dr. Werner Vogels – Giám đốc Công nghệ (CTO) kiêm Phó Chủ tịch, Amazon**

### Thông tin sự kiện

- **Sự kiện:** Fireside Chat with Dr. Werner: Navigating the Future of Cloud & AI in Vietnam
- **Ngày:** Thứ Sáu, 02/10/2026
- **Thời gian:** 08:30 – 12:00
- **Địa điểm:** Tầng 26, Bitexco Financial Tower, TP. Hồ Chí Minh
- **Đơn vị tổ chức:** AWS First Cloud Journey × Amazon

### Nội dung nổi bật

#### Học từ những sự cố vận hành thực tế

Kinh nghiệm của Amazon với một sự cố cơ sở dữ liệu lớn cho thấy tầm quan trọng của việc hiểu cách hệ thống thực sự được sử dụng, thay vì phụ thuộc mù quáng vào các công nghệ sẵn có. Khi phân tích workload thực tế, các kỹ sư nhận thấy 70% thao tác với cơ sở dữ liệu chỉ là các truy vấn key-value đơn giản, và chỉ 10% thực sự cần đến khả năng của cơ sở dữ liệu quan hệ. Nhận định này góp phần tạo ra công nghệ key-value phân tán của Amazon, nền tảng của Dynamo và DynamoDB.

#### Operational excellence

Buổi trò chuyện nhấn mạnh bốn thực hành quan trọng:

- Đo hiệu năng hệ thống ở phân vị P99.9.
- Thiết kế hệ thống với giả định rằng sự cố chắc chắn sẽ xảy ra.
- Tổ chức GameDay để chủ động kiểm thử khả năng phục hồi sau sự cố.
- Tận dụng mô hình pay-as-you-go của cloud để nâng cao trách nhiệm vận hành và chất lượng dịch vụ.

#### Renaissance Developer

Dr. Vogels giới thiệu ý tưởng rằng lập trình viên hiện đại cần nhiều hơn kỹ năng lập trình. Thay vì bị AI thay thế, một Renaissance Developer cần có:

- Sự tò mò và tinh thần học hỏi liên tục
- Tư duy hệ thống
- Tinh thần làm chủ (ownership)
- Chuyên môn hình chữ T (T-shaped)
- Kỹ năng giao tiếp tốt

#### AI là chất xúc tác, không thay thế kỹ sư

AI có thể đẩy nhanh đáng kể việc làm prototype và tự động hóa các công việc lặp lại, nhưng kết quả do AI tạo ra vẫn cần con người đánh giá về bảo mật, các trường hợp biên (edge case) và hiệu quả. Khả năng phán đoán, sự sáng tạo, sự thấu cảm và trách nhiệm nghề nghiệp của con người vẫn rất quan trọng.

### Bài học rút ra

- Hệ thống nên được thiết kế dựa trên cách sử dụng và vấn đề thực tế, thay vì chỉ chọn công nghệ vì nó phổ biến.
- Sự cố là điều cần được lường trước và lên kế hoạch. Hệ thống đáng tin cậy cần có kiểm thử, giám sát và khả năng tự động phục hồi.
- AI thay đổi vai trò của kỹ sư chứ không xóa bỏ vai trò đó. Kỹ sư cần vượt ra ngoài việc chỉ viết code và chịu trách nhiệm cho toàn bộ hệ thống.
- Học hỏi liên tục là điều thiết yếu vì ngôn ngữ lập trình, framework, công nghệ cloud và công cụ AI luôn thay đổi.
- Tư duy hệ thống ngày càng quan trọng, đặc biệt khi làm việc với các hệ thống cloud phân tán.
- Khả năng phán đoán và tinh thần làm chủ của con người vẫn rất quan trọng. Code do AI tạo ra vẫn cần được đánh giá về bảo mật, tính đúng đắn, edge case và hiệu quả.

### Ứng dụng vào công việc

Những bài học từ sự kiện có thể được áp dụng trực tiếp vào dự án FCAJ và quá trình học AWS của tôi:

- **Tư duy hệ thống:** Hiểu cách các dịch vụ AWS phối hợp với nhau thay vì học riêng lẻ từng dịch vụ. Ví dụ, một ứng dụng AWS hoàn chỉnh có thể bao gồm thu thập dữ liệu, lưu trữ, huấn luyện mô hình, suy luận (inference), giám sát và tương tác với người dùng.
- **"Everything Fails All the Time":** Thiết kế ứng dụng với giả định sự cố sẽ xảy ra. Trong dự án của tôi, điều này có thể được áp dụng thông qua giám sát, xử lý lỗi, cô lập dịch vụ và dọn dẹp tài nguyên AWS đúng cách.
- **Renaissance Developer:** Mở rộng ra ngoài việc phát triển mô hình AI/ML, tìm hiểu thêm về hạ tầng cloud, cơ sở dữ liệu, bảo mật, kiến trúc hệ thống và giao tiếp.
- **Sử dụng AI có trách nhiệm:** Công cụ AI giúp phát triển nhanh hơn, nhưng tôi vẫn cần kiểm tra code được tạo ra, hiểu cách nó hoạt động và chịu trách nhiệm cho hệ thống cuối cùng thay vì phụ thuộc mù quáng vào giải pháp do AI tạo ra.

### Trải nghiệm sự kiện

Buổi Fireside Chat mang lại một góc nhìn giá trị về cách một tổ chức công nghệ lớn tiếp cận cloud và AI. Thay vì chỉ tập trung vào công cụ hay công nghệ cụ thể, buổi trò chuyện nhấn mạnh các nguyên tắc kỹ thuật, độ tin cậy của hệ thống, học hỏi liên tục và trách nhiệm của con người.

Khái niệm Renaissance Developer đặc biệt phù hợp với quá trình phát triển của tôi với tư cách một sinh viên AI, vì nó cho thấy để trở thành một kỹ sư giỏi cần cả chiều sâu kỹ thuật lẫn hiểu biết rộng về nhiều lĩnh vực công nghệ khác nhau.

### Một số hình ảnh sự kiện

![Sự kiện 3 – ảnh 1](/images/4-Event/event3_img1.jpg)
