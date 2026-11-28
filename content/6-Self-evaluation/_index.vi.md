---
title: "Tự đánh giá"
date: 2025-09-09
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

Từ **14/09/2026** đến **14/12/2026**, em thực tập trong chương trình First Cloud AI Journey tại **Công ty TNHH Amazon Web Services Việt Nam**, nơi em được đem những gì đã học ở trường vào một dự án thật và một nhóm làm việc thật.  
Em tham gia **CPAWS – Dự báo giá hàng hóa trên AWS**, dự án nhóm năm người, và phụ trách mảng tin tức và chatbot: chatbot Bedrock phía sau `POST /chat`, các tool, prompt và benchmark của nó, cùng CI của repo. Qua đó em cải thiện kỹ năng **kiến trúc serverless trên AWS, hạ tầng dạng code với AWS CDK, xây dựng và đánh giá ứng dụng LLM trên Amazon Bedrock, review code qua pull request và viết tài liệu kỹ thuật**.  

Em cố gắng hoàn thành công việc đúng hạn, tuân thủ quy tắc làm việc của nhóm (pull request có review, khung giờ deploy, không để bí mật trong repo public) và giữ liên lạc thường xuyên với các bạn trong nhóm để phát hiện vấn đề sớm.

Để nhìn lại quá trình thực tập một cách khách quan nhất có thể, em tự đánh giá theo các tiêu chí dưới đây:

| STT | Tiêu chí                            | Mô tả                                                                                            | Tốt | Khá | Trung bình |
| --- | ----------------------------------- | ------------------------------------------------------------------------------------------------ | --- | --- | ---------- |
| 1   | **Kiến thức và kỹ năng chuyên môn** | Hiểu biết về ngành, áp dụng kiến thức vào thực tế, kỹ năng sử dụng công cụ, chất lượng công việc | ✅   | ☐   | ☐          |
| 2   | **Khả năng học hỏi**                | Tiếp thu kiến thức mới, học hỏi nhanh                                                            | ☐   | ✅   | ☐          |
| 3   | **Chủ động**                        | Tự tìm hiểu, nhận nhiệm vụ mà không chờ chỉ dẫn                                                  | ✅   | ☐   | ☐          |
| 4   | **Tinh thần trách nhiệm**           | Hoàn thành công việc đúng hạn, đảm bảo chất lượng                                                | ✅   | ☐   | ☐          |
| 5   | **Kỷ luật**                         | Tuân thủ giờ giấc, nội quy, quy trình làm việc                                                   | ✅   | ☐   | ☐          |
| 6   | **Tính cầu tiến**                   | Sẵn sàng nhận feedback và cải thiện bản thân                                                     | ✅   | ☐   | ☐          |
| 7   | **Giao tiếp**                       | Trình bày ý tưởng, báo cáo công việc rõ ràng                                                     | ✅   | ☐   | ☐          |
| 8   | **Hợp tác nhóm**                    | Làm việc hiệu quả với đồng nghiệp, tham gia nhóm                                                 | ✅   | ☐   | ☐          |
| 9   | **Ứng xử chuyên nghiệp**            | Tôn trọng đồng nghiệp, đối tác, môi trường làm việc                                              | ✅   | ☐   | ☐          |
| 10  | **Tư duy giải quyết vấn đề**        | Nhận diện vấn đề, đề xuất giải pháp, sáng tạo                                                    | ✅   | ☐   | ☐          |
| 11  | **Đóng góp vào dự án/tổ chức**      | Hiệu quả công việc, sáng kiến cải tiến, ghi nhận từ team                                         | ✅   | ☐   | ☐          |
| 12  | **Tổng thể**                        | Đánh giá chung về toàn bộ quá trình thực tập                                                     | ✅   | ☐   | ☐          |

### Điểm mạnh

- **Làm đến nơi phần việc được giao:** Em nhận mảng chatbot từ lúc đề xuất contract `POST /chat` đến khi chatbot chạy trên production: Lambda gọi Claude Haiku trên Amazon Bedrock với tool use, giới hạn lượt hỏi theo ngày, bộ tin Fed/EIA gắn nhãn tay và các bộ benchmark để đo chất lượng câu trả lời.
- **Đo trước khi kết luận:** Mỗi lần đổi prompt hay thử router, em chạy trên bộ câu hỏi đã khóa từ trước và so với ngưỡng đặt sẵn. Những hướng không đạt ngưỡng, như router chuyển câu đơn giản sang model nhỏ hay gắn nhãn tin tự động bằng Bedrock, em ghi lại kết quả rồi chọn cách đơn giản hơn chứ không cố đưa vào sản phẩm.
- **Giữ chất lượng chung của repo:** Em dựng CI chạy test và lint cho mọi pull request ngay từ những tuần đầu của dự án, và thường xuyên review PR của các bạn, nhờ vậy nhiều lỗi được phát hiện trước khi merge.
- **Viết tài liệu cho người sau:** Em ghi lại các quyết định quan trọng của mảng chatbot vào ADR và tài liệu trong repo, kèm kịch bản demo và kịch bản hỏi đáp, để các bạn trong nhóm có thể đọc lại và trình bày thay khi cần.

### Cần cải thiện

- **Nền tảng học máy và thống kê.** Em cần củng cố kiến thức về dự báo chuỗi thời gian và cách đánh giá một mô hình dự báo, để theo kịp và đặt được câu hỏi cho phần mô hình của dự án thay vì chỉ dựa vào các bạn trong nhóm.
- **Kiểm thử với câu hỏi tự nhiên từ sớm.** Một vài lỗi của chatbot, như bỏ sót tin khi câu hỏi không nêu tài sản hay ngày đăng tin lệch một ngày do giờ UTC, chỉ lộ ra khi nhóm hỏi thử trên production sau khi đã đóng băng. Lần sau em muốn đưa những kiểu câu hỏi người dùng hay hỏi vào bộ kiểm thử ngay từ đầu.
- **Chia thời gian giữa dự án và việc học.** Những tuần cuối em dồn sức cho dự án và báo cáo nên học ít nội dung mới. Em cần lên kế hoạch đều hơn để vừa làm sản phẩm vừa tiếp tục học các mảng còn yếu.

### Suy ngẫm

- Trước kỳ thực tập, em nghĩ phần khó nhất khi làm chatbot là chọn model và viết prompt. Khi làm thật, phần tốn công nhất lại là đo: câu trả lời đúng hay sai, đúng đến đâu, và mỗi câu hỏi tốn bao nhiêu tiền.
- Em cũng hiểu rõ hơn vì sao chatbot nên lấy số liệu qua tool thay vì để model tự nhớ. Khi con số đến từ hệ thống và tin tức có nguồn kèm theo, người dùng mới kiểm chứng được câu trả lời, và nhóm mới biết sai ở đâu để sửa.
- Làm một sản phẩm chạy thật trên AWS khiến em để ý đến chi phí ngay từ lúc thiết kế: mỗi câu hỏi đều tốn token, nên giới hạn lượt hỏi, `maxTokens` và việc theo dõi Cost Explorer quan trọng không kém chất lượng câu trả lời. Em xin cảm ơn các anh chị mentor, ban tổ chức chương trình FCAJ và các bạn trong nhóm CPAWS đã hỗ trợ em trong ba tháng thực tập.
