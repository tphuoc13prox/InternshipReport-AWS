---
title: "Web trên Amplify Hosting"
date: 2025-09-09
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Web gồm những gì

Web nằm trong `web/`: React + TypeScript + Vite + Tailwind, gọi API bằng TanStack Query, vẽ biểu đồ bằng Recharts, đăng nhập bằng Amplify UI `Authenticator` trên user pool Cognito có sẵn (không dùng Amplify backend). Từ bản v1.0.0, web có một trang giới thiệu công khai; mọi trang còn lại đều nằm sau đăng nhập. Web cũng có chế độ tối (dark mode). Mỗi lần gọi API, web gắn ID token vào `Authorization`; API trả 401 là web đưa người dùng về màn đăng nhập.

| Trang | Nội dung |
|---|---|
| Trang giới thiệu (công khai) | CPAWS là gì, đăng nhập và đăng ký |
| Trang chủ | Bốn thẻ tài sản |
| Trang tài sản (vàng, bạc, Brent, WTI) | Bảy thẻ kỳ hạn, khối quy đổi ra số tiền, biểu đồ quạt, biểu đồ giá 2 năm, bảng nâng cao, khối "Độ tin cậy", khung chat |
| "Độ chính xác" | Khối độ tin cậy của cả bốn tài sản và cách đọc chỉ số |
| "Giới thiệu" | Cách đọc dự báo, phương pháp, nguồn dữ liệu |

Các quy tắc hiển thị nhóm đã thống nhất:

+ **Luôn là một khoảng kèm xác suất**, không bao giờ chỉ một mức giá. Mô hình dự báo log return; web đổi ra khoảng giá theo giá đóng cửa mới nhất.
+ **Quy tắc 60/40 cho nhãn xu hướng.** Xác suất tăng (đã làm tròn) ≥ 60% là "Nghiêng tăng", ≤ 40% là "Nghiêng giảm", ở giữa là "Chưa rõ xu hướng". Nhãn và con số dùng cùng giá trị làm tròn, nên không bao giờ mâu thuẫn.
+ **Quy đổi VND** theo tỷ giá trong snapshot: vàng, bạc theo VND/lượng (1 lượng = 37,5 g), dầu theo VND/thùng. Nhãn vàng ghi rõ đây là giá thế giới quy đổi, không phải giá vàng miếng trong nước.
+ **Vàng, bạc** được quy đổi từ GLD và SLV, và trang "Giới thiệu" ghi rõ điều này. Giá chỉ hiện sau khi đăng nhập, có dòng nguồn, không có chức năng tải dữ liệu.
+ Trang nào cũng có câu dự báo không phải lời khuyên đầu tư.
+ Biểu đồ lịch sử giá 2 năm có từ bản v1.0.0. Chuỗi giá do forecasts-api đọc từ một file trên S3 rồi trả về, nên trình duyệt không bao giờ gọi thẳng nguồn giá.

<!-- ẢNH: trang tài sản vàng, bảy thẻ kỳ hạn có khoảng 80%, xác suất tăng và nhãn 60/40 -->
[ẢNH: trang tài sản vàng – bảy thẻ kỳ hạn]

#### Bước 1 – Chạy trên máy

Tạo `web/.env.local` với region, ID user pool và ID app client lấy từ `CpawsAuth`, URL API lấy từ `CpawsApi`, và các cờ tính năng:

| Cờ | Tác dụng |
|---|---|
| `VITE_AUTH_ENABLED` | Bắt buộc đăng nhập |
| `VITE_SHOW_METAL_PRICES` | Hiện giá vàng, bạc (tắt: chỉ hiện % thay đổi và xác suất) |
| `VITE_ACCURACY_API` | Gọi `/accuracy/{asset}` và hiện khối "Độ tin cậy" |
| `VITE_ALLOW_SIGNUP` | Hiện màn đăng ký và nhập mã email của `Authenticator` (mặc định tắt) |

```bash
cd web
npm ci
npm run dev
```

Dev server chạy ở `http://localhost:5173`, cũng là `allowed_origin` mặc định của `CpawsApi`. Có thể chạy các bước kiểm giống CI bằng `npm run typecheck`, `npm test` và `npm run build`.

`VITE_ALLOW_SIGNUP` chỉ ẩn hay hiện màn hình. Nó không chặn được ai vì ID app client là giá trị công khai; thứ chặn thật là cấu hình user pool ở [5.5.1](../5.5-api-and-chatbot/5.5.1-cognito-and-api/).

#### Bước 2 – Tạo app Amplify

Nhóm tạo app bằng tay ngày 04/11 để kịp buổi tập dượt, và để việc nối GitHub sau, vì nối GitHub sẽ tạo app và domain mới.

1. Mở **AWS Amplify** → **Create new app** → **Deploy without Git**.
2. Tên app `cpaws-dev-web`, nhánh `dev`.
3. Thêm rewrite để mọi route của SPA trả về `index.html` (source `</^[^.]+$|\.(?!(css|gif|ico|jpg|js|png|txt|svg|woff|woff2|ttf|map|json)$)([^.]+$)/>`, target `/index.html`, type `200 (Rewrite)`).

<!-- ẢNH: Amplify console, app cpaws-dev-web, nhánh dev, nút Deploy updates -->
[ẢNH: Amplify console – app `cpaws-dev-web`, nhánh `dev`]

#### Bước 3 – Build và upload

Vite gắn cứng các giá trị `VITE_*` vào bundle **lúc build**. Muốn đổi cờ thì build và upload lại; API giữ nguyên.

```bash
cd web
npm run build
```

Nén nội dung `web/dist` thành file zip, rồi trên console mở `cpaws-dev-web` → nhánh `dev` → **Deploy updates** và kéo file zip vào. Mở domain, đăng nhập, kiểm trang dự báo và chatbot.

Vì zip upload tay không gắn với commit, nhóm ghi lại mỗi lần deploy dùng commit và cờ nào:

| Ngày | Commit | `VITE_SHOW_METAL_PRICES` | `VITE_ACCURACY_API` |
|---|---|---|---|
| 04/11 | `047189f` | `false` | `false` |
| 07/11 | `c94849a` | `true` | `false` |
| 08/11 | `2e7d3fa` | `true` | `false` |
| 13/11 | `26399d0` | `true` | `true` |
| 22/11 | bản `v1.0.0` (`ba3d9f4`), thêm `VITE_ALLOW_SIGNUP=true` | `true` | `true` |

#### Bước 4 – Trỏ API về domain Amplify

```bash
cdk deploy CpawsApi -c allowed_origin=https://dev.<APP_ID>.amplifyapp.com --profile cpaws-dev
```

Sau đó mở developer tools của trình duyệt, kiểm preflight trả đúng origin Amplify và không có lỗi CORS. Vì API chỉ nhận một origin, web trên Amplify và web trên localhost không gọi API cùng lúc được. Muốn quay lại phát triển trên máy thì deploy lại với `http://localhost:5173`, mất khoảng 3 phút.

**URL Production:** [CẦN BỔ SUNG: link Amplify của nhóm]
