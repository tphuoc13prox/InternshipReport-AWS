---
title: "Sự Kiện 5"
date: 2026-11-14
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

# Báo Cáo Tóm Tắt: "FCAJ x FPTU: Cloud Mastery 1"

### Mục tiêu sự kiện

- Làm rõ khoảng cách giữa một bản prototype AI và một ứng dụng AI vận hành ổn định trên môi trường production.
- Giới thiệu bốn trụ cột của AI trên production với AWS: bảo mật, khả năng mở rộng, độ tin cậy và chi phí.
- Trình bày các case study doanh nghiệp thực tế đang chạy AI agent trên Amazon Bedrock AgentCore.
- Giới thiệu các thành phần chính của Amazon Bedrock AgentCore, gồm Runtime, Identity, Gateway, Memory và Observability.
- Hướng dẫn cách kết hợp Strands Agents SDK với AgentCore để xây dựng và vận hành AI agent.

### Diễn giả

- **Khang Duong – Thành viên, FCAJ**
- **Huyen – AI Engineer Fresher, FPT Software HCMC**
- **Nguyen Lam Anh – AI Engineer, FCAJ**
- **Nguyen Quoc Bao – Cloud Engineer, AWS First Cloud AI Journey**
- **Huynh An Khuong – AI Engineer, FPT Software**

### Thông tin sự kiện

- **Sự kiện:** FCAJ x FPTU: Cloud Mastery 1
- **Ngày:** 14/11/2026
- **Thời gian:** 09:00 – 12:00
- **Địa điểm:** Tầng 26, Bitexco Financial Tower, TP. Hồ Chí Minh
- **Đơn vị tổ chức:** AWS First Cloud AI Journey (FCAJ) và Trường Đại học FPT (FPTU)
- **Vai trò:** Người tham dự

### Nội dung nổi bật

#### Từ prototype đến production

- So sánh AI trong môi trường học thuật với AI trong doanh nghiệp qua hình ảnh căn bếp gia đình và chuỗi nhà hàng: AI doanh nghiệp được đánh giá bằng giá trị kinh doanh chứ không chỉ độ chính xác, phải làm việc với dữ liệu thay đổi liên tục và phục vụ hàng triệu người dùng 24/7 mà không gián đoạn.
- Nhấn mạnh rằng model chỉ là một phần của hệ thống production; bên cạnh đó còn cần data pipeline, giám sát, bảo mật, khả năng mở rộng, độ tin cậy và kiểm soát chi phí.

#### Bốn trụ cột trên AWS

- Bảo mật: AWS KMS để bảo vệ dữ liệu, AWS IAM để kiểm soát truy cập và guardrails để xử lý các rủi ro riêng của AI, chẳng hạn người dùng tìm cách khiến model trả lời sai.
- Khả năng mở rộng và độ tin cậy: AWS Lambda, Auto Scaling và Elastic Load Balancing để đáp ứng lưu lượng thay đổi, kết hợp health check, retry, failover và cảnh báo.
- Tối ưu chi phí bằng pay-per-use, chọn đúng cấu hình và caching, cùng nguyên tắc dùng AI hợp lý: một model lớn đóng vai trò điều phối, các model nhỏ và rẻ hơn xử lý tác vụ đơn giản, và cache lại các câu hỏi lặp lại.

#### Case study doanh nghiệp: Cox Automotive và Mobileye

- Cox Automotive xác định được 57 cơ hội ứng dụng agentic AI nhưng gặp hai vấn đề: agent cần quyền truy cập tối thiểu vào dữ liệu nhạy cảm, và chưa có môi trường chuẩn để chạy agent. Sau khi chuẩn hóa trên AgentCore Runtime, AgentCore Identity và Amazon Bedrock Guardrails, công ty đã đưa 17 giải pháp agent vào production.
- Tại Mobileye, 66% ticket hỗ trợ là các câu hỏi về trạng thái thông thường. Một thiết kế hybrid kết nối hệ thống ticket on-premises với một support agent chạy trên AgentCore Runtime, dùng AWS Secrets Manager và MCP để truy cập các dịch vụ nội bộ, giúp rút thời gian phản hồi ticket từ vài giờ xuống khoảng một phút.
- Phần này kết thúc với ba bài học: đưa bảo mật và khả năng mở rộng vào nền tảng ngay từ đầu thay vì vá sau, agent chỉ tạo ra giá trị khi kết nối được với hệ thống thật và dữ liệu thật, và không thể tin một agent nếu không đo lường được nó.

#### AI trong doanh nghiệp: Amazon Bedrock AgentCore

- Dùng case study của Duolingo để cho thấy lợi ích của một nền tảng agent dùng chung, thay vì mỗi nhóm tự dựng lại cùng một bộ hạ tầng.
- Minh họa cách xây dựng một agent tra cứu đơn hàng bằng Strands Agents SDK và chạy trên AgentCore Runtime, nơi cung cấp sẵn tài nguyên tính toán được quản lý, session tách biệt và khả năng mở rộng.
- Nhấn mạnh rằng dù runtime được quản lý, đội phát triển vẫn chịu trách nhiệm về business rule, dữ liệu, công cụ và việc kiểm thử.

#### Cốt lõi của agent: Identity, Gateway, Memory và Observability

- AgentCore Identity tách biệt xác thực chiều vào (người dùng là ai và có được gọi agent hay không) với xác thực chiều ra (agent được phép truy cập những gì thay mặt người dùng).
- AgentCore Gateway cung cấp cho agent một điểm truy cập thống nhất và an toàn tới nhiều công cụ, API và MCP server; AgentCore Memory cung cấp bộ nhớ ngắn hạn cho cuộc hội thoại hiện tại và bộ nhớ dài hạn cho các thông tin như sở thích người dùng và bản tóm tắt.
- AgentCore Observability, tích hợp với Amazon CloudWatch, theo dõi từng bước của agent, gồm lời gọi model, lời gọi công cụ, độ trễ và lượng token sử dụng, theo đúng tinh thần "không thể sửa thứ mà mình không nhìn thấy".

#### Hỏi đáp, mini quiz và kết thúc chương trình

- Người tham dự đặt câu hỏi cho diễn giả về các chủ đề đã được trình bày.
- Tham gia mini quiz vào cuối sự kiện.
- Điền form phản hồi sự kiện và chụp ảnh lưu niệm cùng diễn giả và người tham dự.

### Bài học rút ra

#### AI sẵn sàng cho production

- Một model chạy được chỉ là điểm khởi đầu; bảo mật, khả năng mở rộng, độ tin cậy và chi phí cần được thiết kế ngay từ đầu.

#### Kiến trúc agent

- Hiểu rõ hơn cách một agent framework như Strands kết hợp với một nền tảng được quản lý như Amazon Bedrock AgentCore.
- Hiểu vì sao identity, quyền truy cập công cụ và bộ nhớ nên được xử lý bằng các thành phần dùng chung, an toàn thay vì tự xây lại trong từng dự án.

#### Bảo mật cho AI agent

- Học cách xử lý riêng quyền truy cập chiều vào và chiều ra, đồng thời chỉ cấp cho mỗi agent đúng những quyền nó cần.

#### Giám sát và đánh giá

- Nhận ra rằng agent thường trả lời sai mà không báo lỗi, vì vậy mỗi lần chạy cần được ghi log, theo dõi và đo lường theo các mục tiêu đặt ra từ trước.

### Ứng dụng vào công việc

- Thêm Amazon Bedrock Guardrails cho chatbot trong dự án CPAWS-commodity-forecasting của nhóm để kiểm tra nội dung người dùng gửi vào và câu trả lời của model trước khi Lambda `/chat` trả kết quả.
- Rà soát IAM role của Lambda `/chat` để hàm chỉ có đúng những quyền cần thiết để gọi model Claude Haiku trên Amazon Bedrock.
- Ghi log độ trễ, lượng token và lỗi của từng request chatbot vào Amazon CloudWatch, đồng thời đặt các mục tiêu đơn giản về thời gian phản hồi và chi phí.
- Bổ sung một bộ câu hỏi kiểm thử nhỏ vào pipeline CI để kiểm tra câu trả lời của chatbot trước mỗi lần deploy.
- Tìm hiểu Strands Agents SDK và AgentCore Runtime nếu sau này chatbot cần gọi công cụ, ví dụ tra cứu kết quả dự báo giá hàng hóa mới nhất.

### Trải nghiệm sự kiện

Tham gia "FCAJ x FPTU: Cloud Mastery 1" giúp em có cái nhìn thực tế về những gì cần làm để đưa một ứng dụng AI từ prototype lên production trên AWS.

Phần đầu tiên rất dễ theo dõi vì bốn trụ cột của AI trên production được giải thích qua ví dụ về một chuỗi nhà hàng. Nhờ đó em nhận ra một model chạy tốt trong bản demo khác rất xa một hệ thống phải luôn an toàn, mở rộng được và có chi phí hợp lý khi phục vụ nhiều người dùng.

Các case study của Cox Automotive và Mobileye đặc biệt có giá trị vì cho thấy kết quả cụ thể của những doanh nghiệp đang chạy AI agent trên production. Chúng cũng cho thấy phần lớn công sức nằm ở bảo mật, tích hợp với hệ thống sẵn có và đo lường, chứ không chỉ ở bản thân model.

Các phần sau về Amazon Bedrock AgentCore giúp em nắm được bức tranh tổng thể về các thành phần của nó, từ Runtime, Identity đến Gateway, Memory và Observability. Vì em phụ trách chatbot trong dự án của nhóm, các nội dung về guardrails, quyền truy cập tối thiểu và observability liên quan trực tiếp đến công việc của em.

Nhìn chung, sự kiện giúp em hiểu sâu hơn cách các AI agent doanh nghiệp được xây dựng và vận hành trên AWS, đồng thời cho thấy tầm quan trọng của bảo mật, khả năng giám sát và thiết kế cẩn thận khi đưa AI vào production.

### Một số hình ảnh sự kiện

![Sự kiện 5 – ảnh 1](/images/4-Event/event5_img1.jpg)

![Sự kiện 5 – ảnh 2](/images/4-Event/event5_img2.jpg)
