---
title: "Blog 4"
date: 2026-11-21
weight: 4
chapter: false
pre: " <b> 3.4. </b> "
---

# Từ SageMaker đến thiết bị: Triển khai mô hình ML xuống edge bằng AWS IoT Greengrass V2

bởi Nguyễn Tiến Long | ngày 21/11/2026 | đăng tại [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Bài gốc trên Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2344828116282147/)

*Bài của thành viên nhóm Nguyễn Tiến Long. Nội dung dưới đây giữ đúng câu chữ của bài đã đăng, chỉ trình bày lại bằng markdown.*

Với AWS IoT Greengrass, thiết bị có thể chạy suy luận machine learning ngay tại chỗ trên dữ liệu nó tự sinh ra, dùng mô hình đã huấn luyện trên cloud. Bài này giới thiệu lại phần "Perform machine learning inference" trong Developer Guide V2 của AWS. Mình chưa chạy thử, nội dung là phần tìm hiểu từ tài liệu.

### 1. Vì sao chạy AI ngay trên thiết bị

Tài liệu nêu hai lợi ích của suy luận tại chỗ: độ trễ thấp và tiết kiệm chi phí. Việc huấn luyện và xử lý phức tạp vẫn do cloud đảm nhận. Mô hình có thể huấn luyện ở bất cứ đâu, ví dụ xây và huấn luyện trên Amazon SageMaker AI, lưu vào một bucket S3, rồi dùng làm artifact trong component để chạy trên thiết bị.

Với robot, trang Guidance for Physical AI for Robotics on AWS cho biết Greengrass có hai vai trò: chạy suy luận ML để điều khiển robot theo thời gian thực, và thu dữ liệu cảm biến để cải thiện mô hình. Trong kiến trúc Physical AI Toolchain của AWS, Greengrass là bước cuối cùng, đưa mô hình đã tối ưu xuống thiết bị NVIDIA Jetson.

### 2. Greengrass V2 và các khái niệm nền

Greengrass là runtime IoT mã nguồn mở chạy trên thiết bị. Nó cho phép xử lý dữ liệu tại chỗ, chạy dự đoán bằng mô hình ML, và vẫn giao tiếp an toàn với AWS IoT Core (Hình 1).

![Tổng quan AWS IoT Greengrass](/images/3-Blog/blog4_figure1.jpg)

*Hình 1: Tổng quan AWS IoT Greengrass, thiết bị chạy phần mềm client Greengrass và kết nối bảo mật với dịch vụ Greengrass trên AWS Cloud. Nguồn: AWS IoT Greengrass Developer Guide.*

Có ba khái niệm cần nhớ. **Core device** là thiết bị đã cài phần mềm Greengrass. **Component** là một đơn vị phần mềm được đóng gói để cài lên thiết bị. **Deployment** là việc đẩy một hoặc nhiều component từ cloud xuống thiết bị. Phần mềm lõi chạy trên thiết bị gọi là **nucleus**.

Điểm khác của V2 là gom mô hình, runtime và mã suy luận thành các component, nên việc cài runtime, tải mô hình và chạy suy luận diễn ra trong một bước. Greengrass V1 đã hết hỗ trợ từ ngày 7/10/2026, vì vậy các hướng dẫn cũ trên mạng (thường viết cho V1) có thể không còn dùng được.

### 3. Suy luận ML hoạt động như thế nào

AWS chia phần ML thành ba loại component:

- **Model:** chứa mô hình ML dưới dạng artifact của Greengrass.
- **Runtime:** chứa script cài framework ML và các thư viện phụ thuộc lên thiết bị.
- **Inference:** chứa mã suy luận, và khai báo hai loại trên làm phụ thuộc.

Quy trình gồm bốn bước: tạo model component, tạo runtime component, tạo inference component, rồi deploy. Khi deploy component suy luận, Greengrass tự động deploy luôn model và runtime mà nó phụ thuộc (Hình 2). AWS cung cấp sẵn các component mẫu cho phân loại ảnh và phát hiện vật thể, nên có thể chạy thử mà chưa cần tự viết gì.

![Luồng suy luận ML với Greengrass](/images/3-Blog/blog4_figure2.jpg)

*Hình 2: Luồng suy luận ML với Greengrass. Sơ đồ tự vẽ dựa trên tài liệu AWS.*

### 4. Mô hình đến từ đâu và điều kiện để chạy

Greengrass dùng mô hình tự huấn luyện được lưu trên Amazon S3. Bucket chứa mô hình cần đáp ứng vài điều kiện: cùng tài khoản AWS và cùng Region với các component, không dùng mã hóa SSE-C (chỉ hỗ trợ SSE-S3 hoặc SSE-KMS), và tên bucket không chứa dấu chấm. Vai trò của thiết bị phải cho phép hành động `s3:GetObject` để Greengrass đọc được mô hình.

Phía thiết bị cần một Greengrass core device và tối thiểu 500 MB dung lượng trống để dùng các component mẫu của AWS. Runtime mẫu gồm Deep Learning Runtime (DLR) và TensorFlow Lite. Mô hình mẫu là ResNet-50 cho phân loại ảnh và YOLOv3 cho phát hiện vật thể.

### 5. Tùy biến theo nhu cầu

Các component mẫu có thể dùng nguyên hoặc thay từng phần. Có thể đổi cấu hình của component suy luận khi deploy. Có thể thay model component mẫu bằng mô hình của riêng mình, với điều kiện mô hình đó được huấn luyện bằng cùng runtime với mô hình mẫu. Muốn dùng mã suy luận riêng thì tự viết inference component, và muốn dùng runtime khác thì tự viết runtime component. Các component mẫu cũng dùng được làm khuôn để viết component của mình.

### 6. Cách bắt đầu

Việc đầu tiên là có một core device. Tutorial *Getting started with AWS IoT Greengrass V2* hướng dẫn cài Greengrass lên một thiết bị Linux (ví dụ Raspberry Pi) hoặc Windows, rồi tạo và deploy một component Hello World, mất khoảng 20 đến 30 phút. Tutorial này chưa có phần ML. Phần suy luận phân loại ảnh mẫu nằm ở tài liệu ML. Tài liệu có hướng dẫn từng lệnh, mình không chép lại ở đây vì bài chỉ giới thiệu.

### 7. Những chỗ cần lưu ý

- SageMaker AI Edge Manager vẫn còn mục riêng trong tài liệu Greengrass, nhưng chính tài liệu ghi dịch vụ này đã bị ngừng từ ngày 26/4/2024. Ai tìm hướng dẫn về nó nên dùng cách tùy biến component ở mục 5.
- Một số component do AWS cung cấp phụ thuộc vào phiên bản nhỏ cụ thể của Greengrass nucleus, nên cần kiểm tra khi nâng cấp nucleus.
- Phần xử lý lỗi của tài liệu liệt kê những lỗi hay gặp khi chạy suy luận: thiếu thư viện, không tìm thấy thiết bị CUDA, lỗi bộ nhớ, hết dung lượng đĩa, hết thời gian chờ.
- Các runtime mẫu trong tài liệu được ghi theo phiên bản cụ thể (DLR v1.6.0 và v1.3.0, TensorFlow Lite v2.5.0), nên nếu dùng thật cần kiểm tra xem còn phù hợp với môi trường của mình không.

### Kết luận và cảm nghĩ

Phần ML trong Greengrass gọn ở chỗ mọi thứ đều là component: mô hình, runtime và mã suy luận tách riêng, ghép lại bằng khai báo phụ thuộc, nên đổi mô hình hay đổi runtime chỉ cần thay đúng một phần.

Khi đọc mình để ý các ví dụ mẫu là phân loại ảnh và phát hiện vật thể nói chung, tài liệu Greengrass không nói riêng về robot. Phần nối với robot đến từ trang Guidance của AWS ở mục 1. Tài liệu cũng còn vài chỗ cũ, như các mục về Edge Manager và những hướng dẫn V1 trên mạng.

Chủ đề này hợp với người làm thiết bị IoT hoặc robot cần suy luận cục bộ vì độ trễ hoặc mạng không ổn định. Với người đang học AWS như mình, nó cho thấy một luồng khá trọn vẹn: huấn luyện trên SageMaker, lưu mô hình trên S3, rồi đẩy xuống thiết bị.

Câu hỏi cho nhóm: có ai từng deploy mô hình lên thiết bị bằng Greengrass chưa, và chọn runtime nào?

### Tài liệu tham khảo

- Tài liệu chính: [Perform machine learning inference (AWS IoT Greengrass V2 Developer Guide)](https://docs.aws.amazon.com/greengrass/v2/developerguide/perform-machine-learning-inference.html)
- [What is AWS IoT Greengrass? (AWS IoT Greengrass V2 Developer Guide)](https://docs.aws.amazon.com/greengrass/v2/developerguide/what-is-iot-greengrass.html)
- [Tutorial: Getting started with AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html)
- [Guidance for Physical AI for Robotics on AWS](https://aws.amazon.com/solutions/physical-ai-for-robotics-on-aws/)
- [Introducing AWS Physical AI Toolchain (AWS Physical AI Blog, 07/10/2026)](https://aws.amazon.com/blogs/physical-ai/introducing-aws-physical-ai-toolchain/)

#AWS #AWSIoTGreengrass #EdgeAI #MachineLearning #IoT #Robotics #AmazonSageMaker #AWSStudyGroup
