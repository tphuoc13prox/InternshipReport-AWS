---
title: "API, xác thực và web"
date: 2026-11-20
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---
#### Các endpoint

REST API stage `dev`, định nghĩa trong `infra/stacks/cpaws_api_stack.py`. Hợp đồng chi tiết nằm ở `docs/api/contract.md`.

| Endpoint | Lambda | Mô tả |
|---|---|---|
| `GET /health` | — | Kiểm tra API, không cần đăng nhập |
| `GET /forecasts` | `forecasts-api` | Dự báo mới nhất của cả 4 tài sản |
| `GET /forecasts/{asset}` | `forecasts-api` | Dự báo mới nhất của một tài sản (`gold`, `silver`, `brent`, `wti`) |
| `GET /accuracy/{asset}` | `accuracy-api` | Độ tin cậy `rolling_8w`, chỉ đọc bảng `cpaws-dev-accuracy` |
| `POST /chat` | `chat-api` | Chatbot (mục [5.5](../../5.5-chatbot/)) |

- **Xác thực:** mọi route trừ `/health` dùng **Cognito authorizer**. Client gửi ID token trong header `Authorization`.
- **Throttling** ở mức stage: 10 request/giây, burst 20.
- **CORS** chỉ cho phép **một** origin, truyền bằng `-c allowed_origin`. Giá trị này được dùng ở ba chỗ: preflight `OPTIONS`, header do Lambda trả về, và `GatewayResponse` cho lỗi 4xx/5xx (để trình duyệt vẫn đọc được mã 401, 429, 504).
- **Mỗi Lambda chỉ có quyền đọc bảng mình cần.** Ví dụ, `forecasts-api` chỉ được `dynamodb:Query` trên bảng `forecasts`.

#### Deploy

```bash
cdk deploy CpawsServing --profile cpaws-dev
cdk deploy CpawsAuth --profile cpaws-dev
cdk deploy CpawsApi --profile cpaws-dev -c allowed_origin=https://<nhanh>.<app-id>.amplifyapp.com
```

Output của `CpawsAuth` có `ForecastsUserPoolId` và `ForecastsUserPoolClientId`. Output của `CpawsApi` có URL của API. Web cần cả ba giá trị này lúc build.

#### Tài khoản người dùng

User Pool `cpaws-dev-users` **không cho tự đăng ký** (`self_sign_up_enabled=False`), nên nhóm tạo tài khoản cho người dùng thử và người chấm:

```bash
aws cognito-idp admin-create-user --profile cpaws-dev \
  --user-pool-id <USER_POOL_ID> --username <EMAIL_NGUOI_DUNG> \
  --user-attributes Name=email,Value=<EMAIL_NGUOI_DUNG> Name=email_verified,Value=true
```

Cognito gửi mật khẩu tạm qua email. Lần đăng nhập đầu, web yêu cầu người dùng đổi mật khẩu.

#### Lấy ID token để thử API

App client cho phép luồng `USER_PASSWORD_AUTH`, nên lấy được token ngay từ CLI:

```bash
read -rs PW   # nhập mật khẩu, không hiện trên màn hình và không vào lịch sử shell
ID_TOKEN=$(aws cognito-idp initiate-auth --profile cpaws-dev \
  --client-id <USER_POOL_CLIENT_ID> --auth-flow USER_PASSWORD_AUTH \
  --auth-parameters USERNAME=<EMAIL_NGUOI_DUNG>,PASSWORD="$PW" \
  --query "AuthenticationResult.IdToken" --output text)
unset PW

API=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev
curl -s -o /dev/null -w "%{http_code}\n" "$API/forecasts/gold"                            # 401
curl -s -o /dev/null -w "%{http_code}\n" -H "Authorization: $ID_TOKEN" "$API/forecasts/gold"  # 200
```

Bản ghi trả về có:
- `base_date`, `model_version` (`baseline-q252`);
- các phân vị theo kỳ hạn h = 1–7, xác suất tăng;
- tỷ giá `fx.usd_vnd` kèm `fx.date`;
- cờ `is_sample` để web gắn nhãn khi đó là dữ liệu mẫu.

#### Web trên Amplify Hosting

Web viết bằng React + TypeScript + Vite trong thư mục `web/`. Các biến `VITE_*` được đọc **lúc build**, nên muốn đổi cờ thì build lại web, không cần deploy API.

| Biến | Ý nghĩa |
|---|---|
| `VITE_API_URL` | URL của API (để trống thì web chạy bằng dữ liệu mẫu) |
| `VITE_AUTH_ENABLED`, `VITE_REGION`, `VITE_USER_POOL_ID`, `VITE_USER_POOL_CLIENT_ID` | Đăng nhập Cognito. ID của User Pool và app client là giá trị công khai của SPA, không phải bí mật |
| `VITE_SHOW_METAL_PRICES` | Hiện giá vàng/bạc quy đổi từ GLD/SLV, kèm dòng ghi nguồn |
| `VITE_ACCURACY_API` | Hiện khối "Độ tin cậy" từ `/accuracy/{asset}` |

```bash
cd web
cp .env.example .env.local   # điền các giá trị ở trên
npm ci
npm run build                # kết quả nằm trong web/dist
```

Nhóm deploy web bằng **zip upload tay**:
1. Nén nội dung `web/dist` thành file zip.
2. Mở Amplify Console, app `cpaws-dev-web`, nhánh `dev`, chọn **Deploy updates** rồi kéo file zip vào.
3. Thêm rewrite để mọi đường dẫn của SPA trả về `index.html`.

Zip upload tay không gắn với commit nào, nên nhóm ghi lại commit của từng lần deploy (bản 13/11/2026 build từ `26399d0`). Repo đã có sẵn `amplify.yml` để sau này nối Amplify với GitHub cho tự build.

Chạy web trên máy với dữ liệu mẫu, không cần AWS:

```bash
cd web && npm ci && npm run dev   # http://localhost:5173
```

<!-- ẢNH: Console API Gateway, cây resource của stage dev (/health, /forecasts, /accuracy, /chat) -->
<!-- ẢNH: Console Cognito User Pool: Sign-up = disabled (che email người dùng) -->
<!-- ẢNH: Console Amplify app cpaws-dev-web, danh sách deployment -->
<!-- ẢNH: Trang đăng nhập của web -->
