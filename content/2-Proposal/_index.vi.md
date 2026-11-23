---
title: "Proposal"
date: "2026-10-21"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CPAWS - Nền tảng dự đoán giá vàng, bạc, dầu thô kèm chatbot giải thích trên AWS
*(Xem dự đoán giá tuần tới theo USD và VND, và hỏi AI vì sao giá có thể tăng hay giảm)*

---

## 1. Tóm tắt chung

Chúng tôi là nhóm **CPAWS** gồm 5 sinh viên thực tập trong chương trình AWS First Cloud AI Journey (FCAJ). Nhóm xây dựng một nền tảng web dự đoán giá **vàng, bạc, dầu Brent và dầu WTI** trong **1–7 ngày tới**, hiển thị theo cả **USD và VND**, kèm **chatbot tiếng Việt trên Amazon Bedrock** giải thích vì sao giá có thể tăng hay giảm dựa trên số liệu của mô hình và tin tức có trích nguồn.

| Thành viên | Vai trò |
|---|---|
| Trịnh Tiến Khải | Research Lead – mô hình dự báo chính, hàm tạo đặc trưng, code Lambda dự đoán |
| Nguyễn Tiến Long | Modeling & Evaluation – mốc so sánh, DLinear, đánh giá độ chính xác |
| Phan Tấn Phước | News & Chatbot – gắn nhãn tin tức, chatbot |
| Châu Trần Mạnh Tiến | Platform – CDK, Lambda, Step Functions, web |
| Huỳnh Ngọc Tuấn | Data & Security Lead – IAM, API Gateway, Cognito, DynamoDB, chi phí |

Hệ thống chạy hoàn toàn trên AWS theo kiến trúc **serverless chạy theo lịch**: thu thập dữ liệu hằng ngày, dự đoán theo lô và lưu sẵn kết quả, nên chi phí thấp và nằm trong khoản **khoảng 200 USD credit AWS** của nhóm. Mục tiêu là một **sản phẩm chạy thật**, không phải bài nghiên cứu.

---

## 2. Phát biểu Vấn đề

### Vấn đề là gì?

Giá vàng, bạc và dầu biến động mạnh khi có sự kiện lớn như xung đột vũ trang, quyết định lãi suất của ngân hàng trung ương hay quyết định sản lượng của OPEC+. Với Việt Nam, đây là các hàng hóa giao dịch chung với thế giới: giá vàng, bạc trong nước bám theo giá quốc tế, còn giá dầu thô là căn cứ chính để tính giá xăng dầu bán lẻ. Người dùng phổ thông dễ xem giá hiện tại nhưng gặp ba khó khăn:

- **Khó biết sắp tới:** dự báo trên báo chí thường là nhận định định tính, không có con số và không công bố độ chính xác trong quá khứ.
- **Khó hiểu vì sao:** tin tức nhiều, rời rạc và không gắn với con số cụ thể.
- **Khó liên hệ với giá trong nước:** giá thế giới tính bằng USD/ounce hay USD/thùng, người dùng khó hình dung mức tác động.

### Giải pháp

CPAWS dự đoán **phân phối tỷ suất sinh lợi (log return) 1–7 phiên tới** dưới dạng 7 phân vị (5%, 10%, 25%, 50%, 75%, 90%, 95%), rồi quy đổi thành khoảng giá và xác suất tăng để hiển thị. Nhóm không dự đoán thẳng mức giá: giá ngày mai gần như bằng giá hôm nay, nên một mô hình chỉ lặp lại giá hôm nay cũng trông như "chính xác 99%". Dự đoán return buộc mô hình học sự thay đổi thật và cho phép đo độ chính xác trung thực.

Chatbot trên Amazon Bedrock chỉ lấy con số từ hệ thống qua **tool use**, mọi lý do đều có trích nguồn, và không đưa ra lời khuyên đầu tư.

| Nhóm người dùng | Nhu cầu chính |
|---|---|
| Người mua vàng tích trữ | Xu hướng tuần tới và khoảng giá dự kiến |
| Doanh nghiệp vận tải, nhập khẩu, người theo dõi giá năng lượng | Xu hướng giá dầu thế giới, căn cứ chính của giá xăng trong nước |
| Sinh viên, người quan tâm đến tài chính | Hiểu yếu tố nào đang tác động lên giá |

### Lợi ích và ROI

- **Con số có kiểm chứng:** khoảng dự đoán và độ chính xác lịch sử được công khai, thay vì nhận định định tính.
- **Giải thích có nguồn:** chatbot gắn mỗi lý do với tin tức hoặc thông cáo chính thức.
- **Gần gũi người dùng Việt:** quy đổi sang VND theo tỷ giá.
- **Chi phí thấp:** không có tài nguyên chạy 24/7, ước tính 15–35 USD/tháng.

---

## 3. Kiến trúc Giải pháp

### Tổng quan

Hệ thống chạy theo lịch: thu thập hằng ngày, dự đoán theo lô rồi lưu sẵn kết quả. Mỗi thành phần chạy độc lập, nên một nguồn dữ liệu gặp lỗi không làm dừng toàn hệ thống.

![Kiến trúc CPAWS](/images/2-Proposal/CPAWS-architecture.jpg)

### Dịch vụ AWS được sử dụng

| Lớp | Dịch vụ AWS | Vai trò |
|---|---|---|
| Thu thập | Amazon EventBridge Scheduler, AWS Lambda | Gọi nguồn dữ liệu theo lịch hằng ngày |
| Lưu trữ | Amazon S3 (data lake), Amazon DynamoDB | Dữ liệu thô, đã làm sạch, đặc trưng, mô hình; kết quả dự đoán và độ chính xác |
| Huấn luyện | AWS Step Functions, Amazon SageMaker AI | Huấn luyện lại, backtest, so sánh champion/challenger |
| Dự đoán | AWS Lambda (container image trên Amazon ECR) | Chạy mô hình đã đóng băng mỗi ngày |
| AI hội thoại | Amazon Bedrock (Converse API, tool use, Guardrails) | Giải thích dự đoán, gắn nhãn tin tức |
| Giao diện | AWS Amplify Hosting, Amazon API Gateway, Amazon Cognito | Web theo tài sản (USD/VND), đăng nhập, giới hạn lượt gọi |
| Vận hành | Amazon CloudWatch, AWS Budgets, AWS CDK, GitHub Actions | Giám sát, cảnh báo chi phí, hạ tầng dạng code, CI/CD |

### Luồng hoạt động

- **Hằng ngày:** EventBridge Scheduler kích hoạt Lambda thu thập giá và tỷ giá vào S3; khi xong, Lambda dự đoán đọc dữ liệu sạch, chạy mô hình champion và ghi 7 phân vị cho h = 1–7 vào DynamoDB.
- **Hằng tuần:** huấn luyện mô hình challenger, backtest trên các tuần gần nhất và chỉ thay champion khi challenger tốt hơn.
- **Khi người dùng hỏi:** chatbot gọi tool để lấy dự đoán, độ chính xác và tin tức liên quan, rồi trả lời kèm trích nguồn.

### Đánh giá theo AWS Well-Architected

| Trụ cột | Vấn đề cần giải quyết | Giải pháp trong CPAWS |
|---|---|---|
| **Operational Excellence** | Vận hành hằng ngày không cần thao tác tay, dễ phát hiện lỗi, thay đổi có kiểm soát | Hạ tầng viết bằng AWS CDK; CI chạy test (pytest, ruff, test hạ tầng) trên mỗi Pull Request; mọi thay đổi cần review, thay đổi IAM/hạ tầng cần người phụ trách bảo mật duyệt; CloudWatch Logs và Alarm cho Lambda; quyết định kiến trúc ghi bằng ADR; image tag theo commit SHA để truy vết |
| **Security** | Bảo vệ dữ liệu và API key, chống lạm dụng API, chống prompt injection | IAM user trong group, đăng nhập phiên ngắn hạn, không có access key dài hạn; mỗi Lambda một IAM role quyền tối thiểu; API key lưu trong SSM Parameter Store (SecureString); API Gateway dùng Cognito authorizer, không cho tự đăng ký; tin tức được đánh dấu là dữ liệu không tin cậy trong prompt, Guardrails giới hạn phạm vi chatbot; repo public không chứa key hay Account ID đầy đủ |
| **Reliability** | Một nguồn dữ liệu lỗi hay thiếu phiên không được làm sai toàn hệ thống | Tách ingest và predict, nối bằng Lambda Destinations cho cả OnSuccess và OnFailure; tài sản lỗi được bỏ qua, các tài sản còn lại vẫn dự đoán; predict không lùi về ngày cũ khi thiếu phiên mới nhất; ingest tải lại 5 phiên gần nhất và ghi đè theo khóa nên chạy lại an toàn; S3 là bản dữ liệu chính, có thể dựng lại bảng DynamoDB từ S3 |
| **Performance Efficiency** | Trả kết quả nhanh cho web và chatbot với chi phí thấp | Dự đoán tính sẵn theo lô; DynamoDB thiết kế theo truy vấn (PK `asset`, SK `base_date`), đọc bản mới nhất bằng một `Query` với `Limit=1`; suy luận bằng NumPy trong Lambda container thay vì PyTorch; dịch vụ serverless tự mở rộng theo lượng truy cập |
| **Cost Optimization** | Chạy cả dự án trong khoảng 200 USD credit | Không có endpoint 24/7, NAT Gateway hay OpenSearch Serverless; DynamoDB on-demand; định tuyến chatbot sang model nhỏ cho câu đơn giản và giới hạn `max_tokens`; Training Job theo lịch, tự tắt; AWS Budgets cảnh báo ở 50 USD/tháng; ước tính chi phí trước khi thêm tài nguyên |
| **Sustainability** | (không chấm) | Chỉ chạy tài nguyên khi cần; region gần người dùng |

---

## 4. Triển khai Kỹ thuật

### Mô hình dự đoán

| Mô hình | Vai trò |
|---|---|
| Naive (return = 0), phân vị thực nghiệm 252 phiên | Mốc so sánh |
| DLinear | Mô hình học máy đầu tiên của bản MVP, nhẹ, chạy trên CPU |
| CEEMDAN + DLinear/LSTM | Mô hình chính: CEEMDAN theo cửa sổ trượt tách chuỗi giá thành thành phần xu hướng và dao động ngắn hạn làm đầu vào |

Tín hiệu ngoài giá gồm chỉ số rủi ro địa chính trị (GPR), chỉ số biến động OVX/GVZ, lịch sự kiện (FOMC, báo cáo tồn kho EIA) và thông cáo chính thức của Fed, EIA. Mọi tín hiệu được căn theo **thời điểm công bố** để tránh rò rỉ dữ liệu tương lai.

### Chia dữ liệu và đánh giá

| Giai đoạn | Thời gian |
|---|---|
| Khởi động CEEMDAN | 2012–2013 |
| Huấn luyện | 2014–2018 |
| Validation (chọn tham số) | 2019 |
| Kiểm tra (chỉ chạy một lần sau khi khóa tham số) | 2020 → nay |

- **Chỉ số chính:** pinball skill so với mốc phân vị 252 phiên.
- **Hiệu chỉnh khoảng dự đoán:** tỷ lệ giá thực rơi vào khoảng 80%, độ rộng khoảng, interval score.
- **Xác suất tăng:** Brier skill.
- **So sánh mô hình:** kiểm định Diebold–Mariano.
- **Không dùng** MAPE hay sai số trên mức giá vì dễ tạo độ chính xác ảo.

### Yêu cầu Kỹ thuật

- **Ngôn ngữ và thư viện:** Python (pandas, NumPy, PyTorch cho huấn luyện, NumPy cho suy luận), React + TypeScript + Vite cho web.
- **Hạ tầng:** AWS CDK (Python), region ap-southeast-1.
- **Quy trình:** GitHub, mỗi việc một nhánh, Pull Request cần ít nhất 1 approval và CI xanh; thay đổi IAM/hạ tầng cần thành viên phụ trách bảo mật review.

---

## 5. Lịch trình & Cột mốc

| Giai đoạn | Hoạt động |
|---|---|
| Trước xây dựng (09–10/2026) | Học nền tảng AWS, chọn đề tài, viết proposal, chuẩn bị dữ liệu giá từ 2012 |
| Tuần 1–2 (12–25/10) | Kiến trúc, dữ liệu sạch, target và cách đánh giá, pipeline thu thập |
| Tuần 3–4 (26/10–08/11) | Mốc so sánh, DLinear MVP, Lambda dự đoán, API và chatbot bản đầu |
| Tuần 5–6 (09–22/11) | Luồng hằng ngày tự động, tín hiệu tin tức, đánh giá; đóng băng tính năng 22/11 |
| Tuần 7 (23–29/11) | Ổn định hệ thống, báo cáo cuối kỳ, tổng duyệt |
| Cuối 11 – đầu 12/2026 | Trình bày trước hội đồng thực tập FCAJ-AWS |

---

## 6. Ước tính Ngân sách

### Chi phí AWS

| Dịch vụ | Ước tính/tháng |
|---|---|
| Lambda, EventBridge Scheduler, Step Functions | < 2 USD |
| S3, DynamoDB (on-demand), ECR | 1–3 USD |
| SageMaker AI Training Job (theo lịch, tắt sau khi chạy) | 3–10 USD |
| Amazon Bedrock (chatbot, gắn nhãn tin) | 5–15 USD |
| API Gateway, Cognito, Amplify Hosting, CloudWatch | 2–5 USD |
| **Tổng** | **khoảng 15–35 USD/tháng** |

### Biện pháp Tối ưu hóa Chi phí

- Không dùng endpoint chạy 24/7, NAT Gateway hay OpenSearch Serverless.
- Dự đoán theo lô bằng Lambda thay vì endpoint suy luận thường trực.
- Định tuyến chatbot theo luật: câu đơn giản dùng model nhỏ, câu phức tạp mới dùng model lớn hơn; giới hạn `max_tokens` theo loại câu.
- AWS Budgets cảnh báo ở mức 50 USD/tháng.

### Kiểm soát Ngân sách

- Toàn bộ chi phí nằm trong khoản credit AWS của nhóm (khoảng 200 USD).
- Mọi tài nguyên mới đều được ước tính chi phí trước khi đề xuất.

---

## 7. Đánh giá Rủi ro

| Rủi ro | Mức | Cách kiểm soát |
|---|---|---|
| Rò rỉ dữ liệu tương lai (CEEMDAN toàn chuỗi, chuẩn hóa sai, tin tức lệch giờ) | Cao | CEEMDAN cửa sổ trượt, backtest walk-forward, căn thời điểm công bố, test tự động |
| Mô hình không thắng được mốc thống kê | Cao | Báo cáo trung thực; sản phẩm dùng mốc thống kê cho tới khi mô hình học máy chứng minh tốt hơn |
| Nguồn dữ liệu thay đổi hoặc giới hạn | Trung bình | Mỗi chuỗi một nguồn chính, có nguồn đối chiếu (EIA); ghi điều khoản vào ADR |
| Chatbot bịa số liệu hoặc bị hiểu là lời khuyên đầu tư | Trung bình | Tool use, trích nguồn, Guardrails, lưu ý bắt buộc |
| Prompt injection qua nội dung tin tức | Trung bình | Đánh dấu tin tức là dữ liệu không tin cậy, Guardrails |
| Chi phí Bedrock tăng do bị lạm dụng | Trung bình | Đăng nhập Cognito, giới hạn lượt gọi, cảnh báo Budgets |
| Phạm vi phình to | Trung bình | Phân mức Must/Should/Could, đóng băng tính năng 22/11 |

---

## 8. Kết quả Dự kiến

### Tiêu chí Thành công

- Sản phẩm web chạy thật trên AWS, tự động thu thập dữ liệu và dự đoán mỗi ngày giao dịch cho 4 tài sản.
- Web hiển thị khoảng dự đoán, xác suất tăng, quy đổi USD/VND và độ chính xác 8 tuần gần nhất.
- Chatbot trả lời bằng tiếng Việt, con số khớp với database, lý do có trích nguồn, từ chối câu hỏi ngoài phạm vi.
- Chi phí nằm trong khoản credit AWS.

### Giá trị Dài hạn

- Khung đánh giá trung thực (mốc thống kê, champion/challenger) có thể dùng lại cho các tài sản khác.
- Pipeline dữ liệu và hạ tầng dạng code giúp mở rộng thêm tính năng dễ dàng.

### Lưu ý về các thay đổi sau proposal

Trong quá trình xây dựng, một số quyết định đã được điều chỉnh và ghi lại bằng ADR: nguồn giá chuyển sang một nguồn thống nhất cho mỗi chuỗi (GLD, SLV, BZ=F, CL=F, VND=X) có đối chiếu với EIA; tin tức chỉ dùng thông cáo chính thức của Fed và EIA; API key lưu trong SSM Parameter Store; bản MVP chưa dùng Knowledge Bases; sản phẩm đang dùng mốc thống kê 252 phiên vì DLinear chưa thắng mốc này trên tập validation.
