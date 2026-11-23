---
title: "Bài học rút ra"
date: 2025-09-09
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

#### Những gì thay đổi giữa proposal và hệ thống

| Trong proposal | Trong hệ thống đang chạy | Lý do |
|---|---|---|
| CEEMDAN + DLinear/LSTM làm mô hình chính | `baseline-q252`, mốc thống kê 252 phiên | 13 cấu hình (DLinear, CEEMDAN + DLinear, co giãn theo biến động, GJR-GARCH + FHS) không vượt được mốc này trên năm validation 2019 |
| Retrain hằng tuần bằng Step Functions + SageMaker | Cửa sổ trượt hằng ngày và champion/challenger có mô hình shadow | Cả champion lẫn challenger đều không có tham số học, nên không có gì để retrain hay host |
| Tin tức gắn nhãn tự động bằng Nova, phục vụ qua Knowledge Base | Tin Fed/EIA gắn nhãn tay, đóng gói cùng Lambda chat | Gắn nhãn tự động không đạt ngưỡng chất lượng |
| Router: câu đơn giản sang Nova, câu phức tạp sang Haiku | Mọi câu đi Haiku | Router đạt 26/30 trên bộ giữ riêng, dưới ngưỡng 90% đã khóa |
| — | Job đo độ tin cậy, shadow S1a, sáu alarm, giới hạn lượt chat | Thêm trong quá trình làm để đo dự báo chạy thật và chặn trần chi phí |

Không mục nào là "thiếu tính năng". Mỗi thay đổi đều đi theo một thí nghiệm có kế hoạch và ngưỡng chốt trước khi chạy.

#### Bài học từ cả dự án

+ **Một mốc đơn giản mà mạnh đáng giá hơn một mô hình phức tạp không vượt được nó.** Với dữ liệu chỉ có giá ở kỳ hạn 1–7 phiên, chiều của return ngắn hạn gần như không đoán được, và giá trị chính của dự báo phân vị nằm ở độ rộng khoảng. Mốc 252 phiên nắm đúng độ rộng đó, nên nhóm đưa nó vào sản phẩm và ghi rõ đây là mốc thống kê, không gọi là mô hình dự báo.
+ **Khóa bài kiểm trước khi nhìn kết quả.** Câu hỏi và code khóa bằng SHA-256, ngưỡng ghi trước, mỗi bộ chỉ chạy chính thức một lần: nhờ vậy nhóm báo cáo được kết quả âm mà không phải tranh cãi lại. Router, prompt v3 và prompt v4 đều bị loại theo đúng cách này.
+ **Thiết kế cho trường hợp lỗi một phần.** Ingest lỗi một nguồn vẫn gọi predict; shadow không chặn được champion; ghi có điều kiện làm mọi lần chạy lại đều an toàn.
+ **Rollback bằng tham số, không phải build lại.** Tag image, công tắc shadow, lịch job accuracy và CORS đều quay lại được bằng một lệnh `cdk deploy -c …`.
+ **Đo khi chạy thật, không chỉ trong backtest.** Job đo độ tin cậy và ngưỡng 20 ngày cho người dùng thấy dự báo đã phát hành thực tế ra sao, và ẩn những con số chưa có ý nghĩa.

#### Điều mình học được từ chatbot

+ **Định hình đầu ra của tool, không chỉ sửa prompt.** Chất lượng tăng nhiều nhất khi đưa cho model ít số hơn và đã tính sẵn (q10/q50/q90 theo %, xác suất đã theo %, không có giá vàng), thay vì thêm quy tắc vào prompt.
+ **Quy tắc quan trọng phải thực thi bằng code.** Trích nguồn do server dựng từ kết quả tool, disclaimer luôn có trong response, thông báo lỗi không lộ tên nội bộ. Model có thể bỏ qua prompt, nhưng không bỏ qua được server.
+ **Thứ gì tăng theo người dùng thì phải có trần cứng.** Giới hạn lượt theo người và cho cả hệ thống biến hóa đơn Bedrock không giới hạn thành một trường hợp xấu nhất biết trước.
+ **Bộ dev gần như không nói lên gì sau khi đã chỉnh theo nó.** Router đạt 83/83 trên bộ câu viết cùng lúc với luật, nhưng chỉ 26/30 trên bộ câu chưa từng thấy.

#### Điểm còn yếu

+ **Vận hành tay.** Một môi trường, deploy từ laptop; giá trị `-c` không được lưu nên dễ quên `allowed_origin` hay `alarm_email`; zip Amplify upload bằng tay.
+ **Alarm không thấy được lượt chạy không diễn ra.** Thiếu dữ liệu được coi là bình thường, nên lịch không kích hoạt chỉ bị phát hiện qua bước kiểm tay.
+ **CORS một origin**, nên localhost và Amplify không gọi API cùng lúc được.
+ **Chất lượng chatbot.** Bộ B vẫn dưới ngưỡng; lỗi còn lại chủ yếu là đọc số. Guardrails chưa gắn.
+ **Dữ liệu.** Vàng, bạc lấy từ GLD, SLV chứ không phải giá giao ngay, và nguồn giá chính là một thư viện không chính thức, có rủi ro điều khoản đã biết.

#### Hướng tiếp theo

+ Alarm trên `Invocations` của ingest, báo khi ba ngày liền không có lượt chạy.
+ Nối Amplify với GitHub để mỗi lần deploy gắn với một commit, và hỗ trợ nhiều origin CORS.
+ Thí nghiệm với tín hiệu ngoài giá (chỉ số biến động, lịch sự kiện, nhãn tin), mỗi thí nghiệm có kế hoạch đăng ký trước.
+ Deploy bản v1.0.1 để chatbot biết ngày hiện tại, và tách môi trường `dev` duy nhất thành môi trường phát triển và production riêng.
+ Gắn Bedrock Guardrails và thêm alarm cho `citation_link_mismatch`.
