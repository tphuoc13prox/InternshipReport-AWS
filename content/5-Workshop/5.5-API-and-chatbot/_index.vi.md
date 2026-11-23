---
title: "API và chatbot"
date: 2025-09-09
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Phần này dựng mọi thứ người dùng đã đăng nhập tương tác: user pool Cognito, REST API với ba Lambda xử lý, và chatbot phía sau `POST /chat`. Đây là luồng 3 trên [sơ đồ kiến trúc](../5.3-architecture/).

| Route | Xác thực | Lambda | Đọc từ |
|---|---|---|---|
| `GET /health` | Không | forecasts-api | — |
| `GET /forecasts`, `GET /forecasts/{asset}` | Cognito | `cpaws-dev-forecasts-api` (256 MB, 10 giây) | `cpaws-dev-forecasts` |
| `GET /accuracy/{asset}` | Cognito | `cpaws-dev-accuracy-api` | `cpaws-dev-accuracy` |
| `POST /chat` | Cognito | `cpaws-dev-chat-api` (512 MB, 28 giây) | `cpaws-dev-forecasts`, `cpaws-dev-chat-usage`, Bedrock |

Deploy `CpawsAuth` trước, sau đó `CpawsApi`. Các bảng ở [5.4.1](../5.4-data-pipeline/5.4.1-storage-and-tables/) phải có sẵn.

#### Nội dung

1. [Cognito và REST API](5.5.1-cognito-and-api/)
2. [Chatbot Bedrock](5.5.2-bedrock-chatbot/)
