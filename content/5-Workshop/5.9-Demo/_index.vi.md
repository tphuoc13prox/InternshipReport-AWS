---
title: "Demo"
date: 2025-09-09
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Đường dẫn

| Mục | Link |
|---|---|
| Web production (Amplify) | [CẦN BỔ SUNG: link Amplify của nhóm] |
| Video demo của nhóm | [CẦN BỔ SUNG: link video của nhóm] |
| Mã nguồn | Xem [5.2](../5.2-prerequiste/) |

Ngoài trang giới thiệu công khai, web nằm sau đăng nhập. Trong thời gian chấm, bạn có thể tự tạo tài khoản bằng nút **Tạo tài khoản** (cần xác minh email). Trang này không công bố bất kỳ thông tin đăng nhập nào.

Video của nhóm trình bày web, pipeline hằng ngày và phần vận hành. Chatbot không nằm trong video; phần này được minh họa bằng ảnh chụp bên dưới.

#### Gợi ý trình tự xem

1. **Tạo tài khoản hoặc đăng nhập** từ trang giới thiệu trên domain Amplify. Mọi trang sau trang giới thiệu đều cần ID token của Cognito.
2. **Trang chủ.** Bốn thẻ tài sản: vàng, bạc, Brent, WTI.
3. **Trang tài sản, ví dụ bạc.** Bảy thẻ kỳ hạn (1–7 phiên). Mỗi thẻ có khoảng dự báo 80% danh nghĩa, xác suất tăng và nhãn 60/40. Bên dưới là biểu đồ giá 2 năm. Chuyển sang chế độ nâng cao để xem bảng q10/q50/q90 và khoảng giá theo từng kỳ hạn.
4. **Quy đổi ra số tiền.** Nhập một số tiền bằng VND và xem khoảng giá trị tương ứng ở kỳ hạn đang chọn.
5. **Khối "Độ tin cậy".** Hiện "Đang thu thập (n/20)" cho tới khi đủ 20 ngày cơ sở được chấm ([5.4.3](../5.4-data-pipeline/5.4.3-rolling-accuracy/)).
6. **Khung chat.** Hỏi về tài sản đang xem. Câu trả lời dùng đúng số trên các thẻ, nêu ngày cơ sở và kỳ hạn, và kết thúc bằng câu disclaimer.
7. **Phía sau.** Trong CloudWatch, log predict lượt 19:15 và sáu alarm ở trạng thái OK.

<!-- ẢNH: trang chủ với bốn thẻ tài sản -->
[ẢNH: trang chủ – bốn thẻ tài sản]

<!-- ẢNH: trang tài sản ở chế độ nâng cao, có bảng q10/q50/q90 -->
[ẢNH: trang tài sản – chế độ nâng cao]

<!-- ẢNH: khối "Độ tin cậy" hiện Đang thu thập (n/20) -->
[ẢNH: khối "Độ tin cậy"]

#### Ví dụ với chatbot

Một vài câu hỏi cho thấy chatbot được thiết kế để làm gì:

| Câu hỏi | Cần để ý |
|---|---|
| "Dự báo bạc 5 phiên tới thế nào?" | Khoảng 80%, trung vị và xác suất tăng từ `get_forecast`; ngày cơ sở; câu nói rõ `baseline-q252` là mốc thống kê 252 phiên; disclaimer |
| "Tuần này có tin gì ảnh hưởng tới dầu WTI không?" | Tin Fed/EIA kèm nguồn, ngày và link, trình bày tách riêng khỏi phần dự báo |
| "Có nên mua vàng bây giờ không?" | Từ chối ngắn gọn: "Mình không đưa ra lời khuyên đầu tư." |

<!-- ẢNH: khung chat, câu trả lời dự báo bạc kèm disclaimer -->
[ẢNH: chat – câu trả lời dự báo bạc]

<!-- ẢNH: khung chat, từ chối câu hỏi xin tư vấn đầu tư -->
[ẢNH: chat – từ chối tư vấn đầu tư]
