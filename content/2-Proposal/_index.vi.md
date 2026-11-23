---
title: "Bản đề xuất"
date: 2025-09-09
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CPAWS – Dự báo giá hàng hóa trên AWS
## Khoảng dự báo 1–7 phiên cho vàng, bạc, dầu Brent và WTI trên kiến trúc serverless, kèm chatbot Bedrock giải thích dự báo

### 1. Tóm tắt điều hành
CPAWS là dự án của nhóm 5 người chúng tôi trong chương trình First Cloud AI Journey. Mục tiêu là làm một website chạy trên AWS: với 4 loại hàng hóa (vàng, bạc, dầu Brent, dầu WTI) và mỗi kỳ hạn từ 1 đến 7 phiên giao dịch tới, website cho xem một khoảng dự báo thay vì một con số giá duy nhất, xác suất giá tăng, và mức độ đáng tin của các dự báo đã phát hành trước đó. Đi kèm là một chatbot dùng Amazon Bedrock, trả lời bằng tiếng Việt và diễn giải dự báo cho dễ hiểu. Chatbot chỉ nói những con số nó đọc được từ hệ thống qua tool, và không đưa lời khuyên đầu tư.

Kế hoạch dựa trên kiến trúc serverless: các Lambda chạy theo lịch thu thập giá và phát hành dự báo mỗi ngày một lần, kết quả lưu ở Amazon S3 và Amazon DynamoDB, người dùng xem qua Amazon API Gateway, Amazon Cognito và web trên AWS Amplify. Toàn bộ hạ tầng viết bằng AWS CDK, chạy trong một môi trường `dev` ở `ap-southeast-1`. Thời gian xây dựng là 7 tuần, từ 12/10 đến 29/11/2026.

### 2. Tuyên bố vấn đề
*Vấn đề hiện tại*

Giá vàng, bạc và dầu thô có thể lên xuống rất mạnh mỗi khi có biến cố lớn: xung đột, quyết định lãi suất, quyết định sản lượng của OPEC+. Ở Việt Nam, các mức giá này ảnh hưởng trực tiếp đến người dân, vì giá vàng, bạc trong nước bám theo giá thế giới, còn giá dầu thô là căn cứ của giá xăng dầu bán lẻ. Ai cũng tra được giá hôm nay, nhưng vẫn còn ba điều khó:

- **Khó biết sắp tới ra sao.** Dự báo trên báo thường là nhận định, không có con số và không ai ghi lại độ chính xác của chúng.
- **Khó biết vì sao giá đổi.** Tin tức nhiều, rời rạc, không gắn với một con số cụ thể.
- **Khó quy ra tiền của mình.** Giá thế giới tính theo USD/ounce hoặc USD/thùng.

*Giải pháp*

Mỗi ngày giao dịch, sau khi phiên Mỹ đóng cửa, một job chạy theo lịch tải giá, làm sạch và lưu một snapshot vào data lake trên S3. Tiếp theo, một hàm khác ước lượng 7 phân vị của **phân phối log return** cho từng kỳ hạn từ 1 đến 7 phiên, rồi ghi kết quả vào DynamoDB. Nhóm cố ý dự báo return chứ không dự báo mức giá: giá ngày mai gần như luôn sát giá hôm nay, nên sai số trên mức giá trông rất nhỏ kể cả với một mô hình chẳng học được gì. Khi dự báo return, mô hình buộc phải nắm được biến động thực sự, và nhóm có thể chấm độ chính xác mà không tự đánh lừa mình.

Web đổi các phân vị ngược lại thành khoảng giá theo USD hoặc VND và xác suất tăng. Vàng và bạc được dự báo trên quỹ ETF GLD và SLV làm chuỗi đại diện, nên giá vàng, bạc trên web là giá quy đổi từ GLD/SLV, không phải giá giao ngay. Chatbot dùng Bedrock Converse API để trả lời câu hỏi tiếng Việt; nó đọc dự báo và tin đã gắn nhãn qua tool chứ không tự đặt ra con số.

*Lợi ích và hoàn vốn đầu tư (ROI)*

- **Với người dùng:** một khoảng giá kèm xác suất phản ánh đúng mức không chắc chắn hơn một "giá mục tiêu", và khối độ tin cậy cho thấy dự báo cũ đã đúng đến đâu.
- **Với nhóm:** dự án đi qua trọn một sản phẩm dữ liệu và ML trên AWS: thu thập, lưu trữ, dự báo, API có xác thực, giao diện web, chatbot GenAI, giám sát và kiểm soát chi phí.
- **Chi phí:** không có gì tính tiền theo giờ. Không có máy chủ chạy cả ngày, không có endpoint mô hình chạy thường trực, không có NAT Gateway, nên chi phí tăng theo mức sử dụng chứ không tăng theo thời gian. Chi tiết ở mục 6.
- CPAWS là dự án học tập, không có doanh thu, nên nhóm không nêu thời gian hoàn vốn. Giá trị của nó là một sản phẩm chạy thật mà người khác có thể đọc và dựng lại từ code CDK.

### 3. Kiến trúc giải pháp

Sơ đồ dưới đây là kiến trúc trong bản đề xuất, nhóm vẽ bằng draw.io lúc bắt đầu dự án. Kiến trúc đang chạy thật nằm ở mục [Workshop](../5-workshop/), còn các điểm khác nhau được liệt kê ở cuối mục này.

![Kiến trúc CPAWS trong bản đề xuất](/images/2-Proposal/CPAWS-architecture.jpg)

*Hình 2.1 – Kiến trúc CPAWS trong bản đề xuất (nhóm CPAWS vẽ bằng draw.io).*

Kế hoạch gồm ba phần:

1. **Pipeline dữ liệu và dự báo hằng ngày** chạy theo lịch: thu thập giá và tin, gắn nhãn tin, lưu vào S3, chạy dự báo và ghi vào DynamoDB.
2. **Vòng huấn luyện lại hằng tuần** do AWS Step Functions điều phối, dùng SageMaker AI training job và model registry.
3. **Dịch vụ cho người dùng:** web trên Amplify, đăng nhập bằng Cognito, REST API trên API Gateway và Lambda, chatbot Bedrock có Guardrails và knowledge base.

*Dịch vụ AWS sử dụng*

- **Amazon EventBridge Scheduler:** khởi động pipeline hằng ngày đúng giờ.
- **AWS Lambda:** thu thập dữ liệu, phát hành dự báo và phục vụ API mà không phải quản lý máy chủ.
- **Amazon S3:** data lake chứa dữ liệu gốc, snapshot giá đã làm sạch và các file đánh giá.
- **Amazon DynamoDB:** lưu dự báo đã tính sẵn để API trả lời nhanh.
- **AWS Step Functions và Amazon SageMaker AI:** dự kiến cho việc huấn luyện lại hằng tuần, training job và model registry.
- **Amazon Bedrock:** gắn nhãn tin và chatbot (Converse API với tool use, Guardrails, Knowledge Bases).
- **Amazon API Gateway và Amazon Cognito:** REST API chỉ người đã đăng nhập mới gọi được, có giới hạn tốc độ request.
- **AWS Amplify Hosting:** chạy web React.
- **Amazon CloudWatch và Amazon SNS:** log, metric, alarm và email cảnh báo.
- **AWS Secrets Manager:** dự kiến lưu API key của nguồn dữ liệu.
- **AWS CDK / AWS CloudFormation:** hạ tầng dạng code, dựng lại hoặc xóa toàn bộ hệ thống bằng một lệnh.

*Thiết kế thành phần*

- **Thu thập dữ liệu:** giá hằng ngày của GLD, SLV, BZ=F, CL=F, tỷ giá USD/VND để hiển thị, cùng tin tức và tín hiệu vĩ mô từ nhiều nguồn.
- **Data lake:** file gốc và snapshot đã làm sạch trên S3, mỗi ngày chạy một snapshot.
- **Dự báo:** 7 phân vị của log return cho h = 1–7 phiên, cho từng tài sản. Bản đề xuất chọn CEEMDAN + DLinear/LSTM làm mô hình chính, các mô hình đơn giản hơn làm mốc so sánh.
- **Phục vụ:** dự báo mới nhất của mỗi tài sản được ghi vào DynamoDB, API chỉ việc đọc.
- **Web:** chế độ cơ bản và nâng cao, giá theo USD hoặc VND, công khai độ chính xác trong quá khứ.
- **Chatbot:** hỏi đáp tiếng Việt về dự báo và tin liên quan, có trích nguồn, từ chối câu ngoài phạm vi và câu xin tư vấn đầu tư.
- **Vận hành:** alarm qua email, ngân sách, CI trên GitHub.

**Những thay đổi sau bản đề xuất.** Trong lúc làm, nhóm kiểm từng phần so với kế hoạch và đổi những gì không đứng vững. Mỗi thay đổi đều có quyết định ghi lại (ADR) trong repo của nhóm:

| Bản đề xuất | Hệ thống đã triển khai | Lý do |
|---|---|---|
| Step Functions → SageMaker training job → model registry, huấn luyện lại mỗi tuần | Hai Lambda nối bằng Lambda destination. Dự báo được phát hành là mốc thống kê 252 phiên `baseline-q252`; một challenger (S1a) chạy song song ở chế độ shadow | 13 cấu hình DLinear, CEEMDAN + DLinear, co giãn theo biến động và GJR-GARCH đều không vượt mốc trên tập validation 2019, nên không có mô hình học nào cần huấn luyện lại |
| Gắn nhãn tin bằng Bedrock batch + Knowledge Bases | Tin gắn nhãn tay, đóng gói dạng JSON cùng Lambda chat | Gắn nhãn tự động không đạt ngưỡng chất lượng |
| Nhiều nguồn tin và tín hiệu vĩ mô | Giá và tỷ giá lấy từ Yahoo Finance (yfinance); EIA và NBP chỉ để đối chiếu | Phạm vi dữ liệu và điều khoản sử dụng |
| Secrets Manager | SSM Parameter Store SecureString | Chỉ cần một key (EIA) |
| Chatbot có Guardrails | Converse + tool use, không bật Guardrails | Bật sau lần benchmark chính thức sẽ làm đổi hành vi của bản đã được đánh giá |
| GitHub Actions CI/CD | Chỉ có CI; deploy chạy từ máy của thành viên | GitHub không giữ quyền AWS và repo để public |
| Amplify nối với Git | Upload zip thủ công | Phải kịp ngày demo; domain giữ nguyên cho người chấm |
| — | Thêm: job độ tin cậy chạy hằng ngày, `GET /accuracy/{asset}`, 6 alarm, giới hạn lượt chat | Cần để đo dự báo chạy thật và kiểm soát chi phí |

### 4. Triển khai kỹ thuật
*Các giai đoạn triển khai*

Nhóm chia 7 tuần thành bốn giai đoạn:

1. **Dữ liệu và mốc so sánh (từ 12/10):** thu thập giá, làm sạch theo lịch giao dịch của sàn, data lake trên S3, và mốc 252 phiên làm chuẩn mà mọi mô hình phải vượt.
2. **Thí nghiệm mô hình (tháng 10 đến đầu tháng 11):** mỗi thí nghiệm commit kế hoạch, danh sách ứng viên và tiêu chí đạt trước khi chạy, và mỗi lần nhìn vào tập validation 2019 đều được đếm.
3. **Sản phẩm (cuối tháng 10 đến giữa tháng 11):** API dự báo, đăng nhập Cognito, web trên Amplify, chatbot `POST /chat`, job độ tin cậy và các alarm. Các mốc: dự báo tự động đầu tiên 27/10, chatbot lên AWS 02/11, web lên Amplify 04/11, challenger shadow 04/11, bật lịch job độ tin cậy 12/11.
4. **Đóng băng và nộp bài (22/11 đến 29/11):** không thêm tính năng sau 22/11; chụp ảnh, tập dượt, viết báo cáo và workshop.

*Yêu cầu kỹ thuật*

- Python 3.12 cho code Lambda và xử lý dữ liệu; AWS CDK (Python) cho hạ tầng; Docker để build image container của ingest và predict; Node.js với React, TypeScript, Vite cho web.
- `pandas_market_calendars` cho lịch giao dịch NYSE và CME, để "h phiên tới" đếm đúng phiên giao dịch thật.
- Quyền dùng model Amazon Bedrock ở `ap-southeast-1` (Claude Haiku 4.5 qua global cross-Region inference profile).
- Không dùng thông tin tương lai: lúc train và lúc dự báo thật áp cùng một quy tắc thời điểm công bố dữ liệu, có test kiểm rằng dự báo tại ngày t không đổi khi sửa giá sau ngày t.
- Chạy lại an toàn: dự báo ghi vào DynamoDB bằng conditional write nên bản đã phát hành không bị ghi đè.
- GitHub Actions chạy `ruff`, `pytest` (gồm cả test synth CDK) và typecheck, test, build của web ở mọi push và pull request.

### 5. Lộ trình & Mốc triển khai

| Thời gian | Công việc |
|---|---|
| 12/10 – 25/10 | Pipeline dữ liệu, mốc so sánh, CI, hợp đồng API, các thí nghiệm mô hình đầu tiên |
| 26/10 – 08/11 | Dự báo tự động đầu tiên (27/10), chatbot trên AWS (02/11), web trên Amplify (04/11), challenger shadow (04/11), demo chatbot nội bộ (08/11) |
| 09/11 – 21/11 | Job và API độ tin cậy (10–12/11), các lần đánh giá chính thức của chatbot và mô hình, viết các chương báo cáo |
| 22/11 | Đóng băng tính năng |
| 23/11 – 29/11 | Kiểm tra cuối, quay video demo, nộp báo cáo và workshop |

### 6. Ước tính ngân sách

Dự toán chi phí cho cả hệ thống dùng bảng AWS Pricing Calculator của nhóm: [CẦN XÁC NHẬN: link AWS Pricing Calculator].

*Chi phí hạ tầng*

- **Khung ngân sách:** khoảng 200 USD credit AWS, AWS Budgets đặt 50 USD/tháng, Cost Anomaly Detection gửi email khi chi phí bất thường.
- **Chi phí theo dịch vụ (từ Pricing Calculator):** [CẦN XÁC NHẬN: bảng chi phí theo dịch vụ từ dự toán Pricing Calculator của nhóm]
- **Chi phí thực tế đến nay:** khoảng 5,28 USD từ 12/10 đến 11/11/2026, được credit chi trả. Dự báo cho giai đoạn 12/11–29/11 khoảng 1,14 USD.
- **Khoản thay đổi lớn nhất là chatbot:** khoảng 0,012 USD mỗi câu hỏi với Claude Haiku 4.5. Giới hạn 100 câu mỗi ngày cho cả hệ thống chặn chi phí này ở khoảng 1,2 USD/ngày, tức khoảng 36 USD/tháng trong trường hợp xấu nhất, vẫn dưới ngân sách 50 USD.
- **Các lựa chọn tiết kiệm:** dự báo theo lô thay vì endpoint thời gian thực, Lambda không chạy trong VPC (không cần NAT Gateway), DynamoDB on-demand, log giữ 1 tháng, mỗi repo ECR giữ tối đa 5 image.

### 7. Đánh giá rủi ro
*Ma trận rủi ro*

| Rủi ro | Ảnh hưởng | Khả năng xảy ra |
|---|---|---|
| Yahoo Finance / yfinance chặn truy cập hoặc điều khoản không cho phép cách nhóm sử dụng | Cao | Trung bình |
| Không mô hình nào vượt được mốc 252 phiên | Trung bình | Cao |
| Chatbot đọc sai con số hoặc trượt sang tư vấn đầu tư | Cao | Trung bình |
| Chi phí Bedrock tăng theo lượng người dùng | Trung bình | Thấp |
| Deploy tay từ một máy làm hỏng một cấu hình (origin CORS, email nhận alarm) | Trung bình | Trung bình |
| File đánh giá bị ghi đè | Trung bình | Thấp |
| Lịch chạy hằng ngày không chạy mà không ai biết | Trung bình | Thấp |

*Chiến lược giảm thiểu*

- **Nguồn dữ liệu:** ghi nguồn trên web, không có chức năng tải dữ liệu giá, giữ sẵn cờ `VITE_SHOW_METAL_PRICES=false` để ẩn giá kim loại.
- **Mô hình:** giữ mốc trong sản phẩm, chỉ đổi mô hình phát hành khi đạt tiêu chí đã đăng ký trước và có ADR.
- **Chatbot:** con số chỉ lấy từ tool; danh sách trích nguồn do server dựng; giới hạn lượt hỏi theo người dùng và cho cả hệ thống; benchmark có bước kiểm tra chính sách.
- **Chi phí:** AWS Budgets, Cost Anomaly Detection và giới hạn lượt chat.
- **Deploy:** luôn chạy `cdk diff` trước, mỗi lần một stack, mỗi lần một người.
- **An toàn dữ liệu:** bật versioning cho S3 (giữ bản cũ 30 ngày), conditional write, explicit deny quyền xóa artifact đánh giá.
- **Lượt chạy bị bỏ lỡ:** kiểm tay sau mỗi lượt 19:15 cho tới khi có alarm "không chạy".

*Kế hoạch dự phòng*

Mọi thay đổi có rủi ro đều hoàn tác được bằng một lệnh deploy hoặc một lần upload web, không cần build lại image: tắt mô hình shadow, quay về image predict hoặc ingest cũ, tắt lịch job độ tin cậy, đưa CORS về `localhost`, hoặc upload lại bản web trước. Rollback không làm mất dữ liệu vì bucket data lake có versioning, còn bảng dự báo và User Pool được giữ lại (`RETAIN`) khi stack thay đổi.

### 8. Kết quả kỳ vọng
*Cải tiến kỹ thuật*

- Dự báo tự động phát hành từ thứ Ba đến thứ Bảy lúc 19:15 giờ Việt Nam, không cần thao tác tay.
- Khối độ tin cậy chỉ tính từ những dự báo đã thực sự phát hành, và chỉ hiện số khi đủ 20 ngày gốc.
- Chatbot có con số luôn khớp trang dự báo, và trích được nguồn tin.

*Giá trị dài hạn*

- Một mẫu serverless chi phí thấp, dựng lại được, cho sản phẩm dữ liệu + ML + GenAI trên AWS.
- Các bước tiếp theo đã rõ: tín hiệu ngoài dữ liệu giá, chỉ đổi mô hình phát hành qua ADR mới, bật Bedrock Guardrails, router chọn model để giảm chi phí chatbot, alarm cho lượt chạy bị bỏ lỡ, tách môi trường `dev`/`prod` với pipeline deploy, và biểu đồ lịch sử giá nếu nguồn dữ liệu cho phép.
