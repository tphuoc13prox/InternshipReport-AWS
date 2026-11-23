---
title: "API, authentication and web"
date: 2026-11-20
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---
#### Endpoints

REST API stage `dev`, defined in `infra/stacks/cpaws_api_stack.py`. The detailed contract is in `docs/api/contract.md`.

| Endpoint | Lambda | Description |
|---|---|---|
| `GET /health` | — | API health check, no sign-in required |
| `GET /forecasts` | `forecasts-api` | Latest forecast for all 4 assets |
| `GET /forecasts/{asset}` | `forecasts-api` | Latest forecast for one asset (`gold`, `silver`, `brent`, `wti`) |
| `GET /accuracy/{asset}` | `accuracy-api` | `rolling_8w` reliability, read-only access to table `cpaws-dev-accuracy` |
| `POST /chat` | `chat-api` | Chatbot (section [5.5](../../5.5-chatbot/)) |

- **Authentication:** every route except `/health` uses a **Cognito authorizer**. The client sends the ID token in the `Authorization` header.
- **Throttling** at stage level: 10 requests/second, burst 20.
- **CORS** allows **one** origin, passed with `-c allowed_origin`. The value is used in three places: the `OPTIONS` preflight, the headers returned by the Lambdas, and the `GatewayResponse` for 4xx/5xx errors (so the browser can still read 401, 429 and 504 codes).
- **Each Lambda can only read the table it needs.** For example, `forecasts-api` only has `dynamodb:Query` on the `forecasts` table.

#### Deploy

```bash
cdk deploy CpawsServing --profile cpaws-dev
cdk deploy CpawsAuth --profile cpaws-dev
cdk deploy CpawsApi --profile cpaws-dev -c allowed_origin=https://<branch>.<app-id>.amplifyapp.com
```

The `CpawsAuth` outputs include `ForecastsUserPoolId` and `ForecastsUserPoolClientId`. The `CpawsApi` outputs include the API URL. The web app needs all three values at build time.

#### User accounts

User Pool `cpaws-dev-users` **does not allow self sign-up** (`self_sign_up_enabled=False`), so the team creates accounts for trial users and graders:

```bash
aws cognito-idp admin-create-user --profile cpaws-dev \
  --user-pool-id <USER_POOL_ID> --username <USER_EMAIL> \
  --user-attributes Name=email,Value=<USER_EMAIL> Name=email_verified,Value=true
```

Cognito emails a temporary password. On first sign-in, the web app asks the user to change it.

#### Getting an ID token to test the API

The app client allows the `USER_PASSWORD_AUTH` flow, so a token can be obtained from the CLI:

```bash
read -rs PW   # type the password; it is not echoed and does not enter shell history
ID_TOKEN=$(aws cognito-idp initiate-auth --profile cpaws-dev \
  --client-id <USER_POOL_CLIENT_ID> --auth-flow USER_PASSWORD_AUTH \
  --auth-parameters USERNAME=<USER_EMAIL>,PASSWORD="$PW" \
  --query "AuthenticationResult.IdToken" --output text)
unset PW

API=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/dev
curl -s -o /dev/null -w "%{http_code}\n" "$API/forecasts/gold"                            # 401
curl -s -o /dev/null -w "%{http_code}\n" -H "Authorization: $ID_TOKEN" "$API/forecasts/gold"  # 200
```

The returned record contains:
- `base_date`, `model_version` (`baseline-q252`);
- quantiles for each horizon h = 1–7 and the probability of a rise;
- the exchange rate `fx.usd_vnd` with `fx.date`;
- the `is_sample` flag, so the web app labels sample data.

#### Web app on Amplify Hosting

The web app is written in React + TypeScript + Vite under `web/`. The `VITE_*` variables are read **at build time**, so changing a flag means rebuilding the web app, not redeploying the API.

| Variable | Meaning |
|---|---|
| `VITE_API_URL` | API URL (leave empty to run on sample data) |
| `VITE_AUTH_ENABLED`, `VITE_REGION`, `VITE_USER_POOL_ID`, `VITE_USER_POOL_CLIENT_ID` | Cognito sign-in. The User Pool and app client IDs are public values of an SPA, not secrets |
| `VITE_SHOW_METAL_PRICES` | Show gold/silver prices converted from GLD/SLV, with a source line |
| `VITE_ACCURACY_API` | Show the "Reliability" block from `/accuracy/{asset}` |

```bash
cd web
cp .env.example .env.local   # fill in the values above
npm ci
npm run build                # output goes to web/dist
```

The team deploys the web app by **manual zip upload**:
1. Zip the contents of `web/dist`.
2. Open the Amplify Console, app `cpaws-dev-web`, branch `dev`, choose **Deploy updates** and drop in the zip file.
3. Add a rewrite so every SPA path returns `index.html`.

A manual zip upload is not tied to any commit, so the team records the commit of each deployment (the 13/11/2026 build is from `26399d0`). The repository already contains `amplify.yml` so Amplify can later be connected to GitHub for automatic builds.

Run the web app locally on sample data, without AWS:

```bash
cd web && npm ci && npm run dev   # http://localhost:5173
```

<!-- ẢNH: Console API Gateway, cây resource của stage dev (/health, /forecasts, /accuracy, /chat) -->
<!-- ẢNH: Console Cognito User Pool: Sign-up = disabled (che email người dùng) -->
<!-- ẢNH: Console Amplify app cpaws-dev-web, danh sách deployment -->
<!-- ẢNH: Trang đăng nhập của web -->
