---
title: "Cognito và REST API"
date: 2025-09-09
weight: 1
chapter: false
pre: " <b> 5.5.1 </b> "
---

#### Bước 1 – User pool (`CpawsAuth`)

`CpawsAuth` tạo user pool `cpaws-dev-users` và một app client cho ứng dụng single-page:

+ email được xác minh tự động bằng mã gửi về địa chỉ đó;
+ mật khẩu tối thiểu 8 ký tự, có chữ hoa, chữ thường, số và ký tự đặc biệt;
+ app client `cpaws-dev-app` không có client secret (trình duyệt không giữ được bí mật), dùng luồng SRP và user-password, không OAuth, bật `prevent_user_existence_errors` để form đăng nhập không lộ email nào đã có tài khoản;
+ `RETAIN`, xóa stack không xóa người dùng.

```bash
cdk diff CpawsAuth --profile cpaws-dev
cdk deploy CpawsAuth --profile cpaws-dev
```

Hai output `ForecastsUserPoolId` và `ForecastsUserPoolClientId` dùng khi build web ở [5.6](../../5.6-web-amplify/). Ứng dụng chạy trên trình duyệt nào cũng phải để lộ hai giá trị này, nên chúng không được coi là bí mật.

**Tự đăng ký.** Bình thường tài khoản do nhóm tạo. Trong thời gian chấm, ADR-0014 mở tự đăng ký có xác minh email. Thay đổi này đã deploy lên `CpawsAuth` ngày 22/11/2026; CloudFormation sửa ba thuộc tính ngay trên pool hiện có, không tạo pool mới nên không mất người dùng. Web cũng được build lại với `VITE_ALLOW_SIGNUP=true` cùng đợt. Theo kế hoạch, tự đăng ký sẽ đóng lại trong vòng một ngày sau khi mentor báo chấm xong, chậm nhất ngày 10/12/2026: tắt trong `CpawsAuth` trước, rồi build web với cờ tắt, rồi disable các user tự tạo ([5.10](../../5.10-cleanup/)).

Muốn tự tạo người dùng thử (dùng email của bạn, không commit vào repo):

```bash
aws cognito-idp admin-create-user --user-pool-id <USER_POOL_ID> \
  --username <TEST_EMAIL> --profile cpaws-dev
```

<!-- ẢNH: Cognito console, user pool cpaws-dev-users, tab Sign-in experience (không lộ email người dùng) -->
[ẢNH: Cognito console – user pool `cpaws-dev-users`]

#### Bước 2 – REST API (`CpawsApi`)

Hệ thống dùng một REST API với stage `dev`. Vài cấu hình cần biết trước khi deploy:

+ **Cognito authorizer** trên mọi route trừ `/health`. API chỉ tin ID token của `cpaws-dev-users`, gửi trong header `Authorization`.
+ **Throttling theo stage** 10 request/giây, burst 20. Đây là lớp chặn chi phí đầu tiên cho chatbot; giới hạn lượt theo người dùng ở [5.5.2](../5.5.2-bedrock-chatbot/) là lớp thứ hai.
+ **CORS cho đúng một origin**, truyền bằng `-c allowed_origin`. Giá trị đó được đặt ở ba nơi: preflight `OPTIONS`, header trong response của các Lambda, và các gateway response lỗi, nhờ vậy khi gặp 401, 429 hay 504 trình duyệt vẫn đọc được mã lỗi.

Deploy với origin sẽ gọi API. Khi phát triển trên máy là `http://localhost:5173`; sau bước [5.6](../../5.6-web-amplify/) là domain Amplify:

```bash
cdk diff CpawsApi -c allowed_origin=ORIGIN_WEB --profile cpaws-dev
cdk deploy CpawsApi -c allowed_origin=ORIGIN_WEB --profile cpaws-dev
```

{{% notice warning %}}
CDK không nhớ giá trị `-c` giữa các lần deploy. Chỉ cần một lần quên `allowed_origin` là origin lặng lẽ quay về `http://localhost:5173` và web trên Amplify hỏng. Lần nào cũng phải ghi kèm.
{{% /notice %}}

Output `ForecastsApiUrl` là URL gốc cho web và cho các lệnh kiểm tra dưới đây.

#### Bước 3 – Kiểm tra route

```bash
curl -sS "$API_URL/health"
curl -sS "$API_URL/forecasts/gold"                              # mong đợi 401
curl -sS "$API_URL/forecasts/gold" -H "Authorization: $ID_TOKEN" # mong đợi 200
```

Để token không nằm trong lịch sử shell, dán bằng `read -rs ID_TOKEN && export ID_TOKEN`.

Một bản ghi dự báo gồm bảy phân vị q05…q95 của log return cho từng kỳ hạn, xác suất tăng, ngày cơ sở, `model_version` (`baseline-q252`) và cờ dữ liệu mẫu hoặc dữ liệu cũ. Web đổi q10–q90 thành khoảng dự báo 80% danh nghĩa.

<!-- ẢNH: terminal, /health 200 rồi /forecasts/gold 401 khi không có token và 200 khi có token (che token) -->
[ẢNH: terminal – `/health` 200, `/forecasts/gold` 401 rồi 200]
