---
title: "Tổng quan workshop"
date: 2025-09-09
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Bài toán

Giá vàng, bạc và dầu thô phản ứng rất nhanh với chiến sự, quyết định lãi suất hay quyết định sản lượng của OPEC+. Ở Việt Nam, giá vàng, bạc trong nước chạy theo giá thế giới, còn giá dầu thô ảnh hưởng tới giá xăng dầu bán lẻ, nên nhiều người quan tâm giá sắp tới sẽ đi đâu. Giá hôm nay thì dễ tra. Cái còn thiếu là một dự báo có con số, cho thấy mức không chắc chắn của nó, và có lịch sử đúng/sai để người dùng kiểm tra.

#### CPAWS làm gì

Với mỗi tài sản và mỗi kỳ hạn h = 1–7 phiên giao dịch, website CPAWS cho xem:

+ **khoảng dự báo**: phân vị 10%, 50% và 90% của return, quy đổi ra khoảng giá theo USD hoặc VND;
+ **xác suất tăng**;
+ **khối độ tin cậy** chấm điểm chính những dự báo hệ thống đã phát hành, khi đã có đủ dự báo tới hạn.

Một **chatbot** tiếng Việt chạy trên Amazon Bedrock giải thích dự báo và liệt kê các tin Fed/EIA liên quan kèm link. Mọi con số chatbot nói đều lấy từ hệ thống qua tool, và chatbot từ chối đưa lời khuyên đầu tư.

| Tài sản | Chuỗi dùng để dự báo | Ghi chú |
|---|---|---|
| Vàng | GLD (ETF nắm giữ vàng thật) | Chuỗi đại diện; giá vàng trên web quy đổi từ GLD, không phải giá giao ngay |
| Bạc | SLV (ETF nắm giữ bạc thật) | Chuỗi đại diện; giá bạc quy đổi từ SLV |
| Brent | BZ=F (hợp đồng tương lai gần nhất) | Có bước nhảy vào ngày chuyển hợp đồng |
| WTI | CL=F (hợp đồng tương lai gần nhất) | Có bước nhảy vào ngày chuyển hợp đồng |

#### Vì sao dự báo return chứ không dự báo giá

Hệ thống dự báo phân phối của **log return** sau h phiên dưới dạng 7 phân vị (q05 đến q95), không dự báo thẳng mức giá. Giá ngày mai thường rất gần giá hôm nay, nên một "mô hình" chỉ lặp lại giá hôm nay đã trông gần như hoàn hảo nếu đo sai số trên mức giá. Dự báo return xóa đi ảo giác đó: mô hình chỉ được điểm cao khi thực sự học được điều gì đó về phần thay đổi. Sau đó web quy đổi phân vị ngược lại thành giá.

Khoảng từ q10 đến q90 được gọi là **khoảng dự báo 80% danh nghĩa**. Tỷ lệ giá thật rơi vào khoảng này được đo riêng (`coverage_80`) và có thể khác 80%.

#### Đang chạy gì trong production

Dự báo phát hành cho cả 4 tài sản là `baseline-q252`, tức **mốc thống kê 252 phiên**: với mỗi h, lấy phân vị thực nghiệm của các return h phiên đã hoàn tất trong 252 phiên gần nhất. Mốc này không có tham số học. Nhóm đã thử 13 cấu hình mô hình khác và không cấu hình nào vượt mốc theo tiêu chí chốt trước mỗi thí nghiệm (chi tiết ở [5.11](../5.11-lessons-learned/)). Một challenger tên S1a chạy song song ở chế độ shadow; người dùng không thấy kết quả của nó.

#### Thành viên nhóm

| Thành viên | Vai trò | Phụ trách chính |
|---|---|---|
| Trịnh Tiến Khải | Research Lead | Mô hình, dữ liệu, Lambda predict, điều phối báo cáo |
| Nguyễn Tiến Long | Modeling & Evaluation | Đánh giá, champion/challenger |
| Phan Tấn Phước | News & Chatbot | Gắn nhãn tin, chatbot |
| Châu Trần Mạnh Tiến | Platform | CDK, Lambda, web, deploy |
| Huỳnh Ngọc Tuấn | Data & Security Lead | IAM, API Gateway, Cognito, DynamoDB, chi phí |

Mã nguồn của nhóm để public; link ở mục [5.2](../5.2-prerequiste/).

#### Bạn sẽ dựng được gì

Cuối workshop, tài khoản của bạn sẽ chạy đủ 5 stack CDK giống nhóm, ở `ap-southeast-1`:

+ một lịch chạy ingest → predict từ thứ Ba đến thứ Bảy lúc 19:15 giờ Việt Nam;
+ 4 bảng DynamoDB và một data lake trên S3;
+ một REST API có Cognito bảo vệ, gồm `GET /forecasts/{asset}`, `GET /accuracy/{asset}` và `POST /chat`;
+ web chạy trên Amplify Hosting;
+ 6 alarm CloudWatch gửi email qua SNS.

Đăng ký tài khoản trên web: trong thời gian chấm, nhóm mở tự đăng ký có xác minh email (ADR-0014, deploy ngày 22/11/2026) và sẽ đóng lại sau khi chấm xong. Chi tiết ở [5.5.1](../5.5-api-and-chatbot/5.5.1-cognito-and-api/).
