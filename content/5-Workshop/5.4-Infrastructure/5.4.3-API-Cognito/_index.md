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
| `GET /forecasts/{asset}` | `forecasts-api` | Latest forecast for one asset (`gold`, `silver`, `brent`, `wti`); add `?history=N` (N = 1–520) to get the closing prices of the last N sessions |
| `GET /accuracy/{asset}` | `accuracy-api` | `rolling_8w` reliability, read-only access to table `cpaws-dev-accuracy` |
| `POST /chat` | `chat-api` | Chatbot (section [5.5](../../5.5-chatbot/)) |

- **Authentication:** every route except `/health` uses a **Cognito authorizer**. The client sends the ID token in the `Authorization` header.
- **Throttling** at stage level: 10 requests/second, burst 20.
- **CORS** allows **one** origin, passed with `-c allowed_origin`. The value is used in three places: the `OPTIONS` preflight, the headers returned by the Lambdas, and the `GatewayResponse` for 4xx/5xx errors (so the browser can still read 401, 429 and 504 codes).
- **Each Lambda can only read the table it needs.** For example, `forecasts-api` only has `dynamodb:Query` on the `forecasts` table and `s3:GetObject` on exactly the `serving/price-history/*` prefix.

#### Two years of price history (`?history=`)

Since `v1.0.0`, the chart on the product page can show up to 520 sessions (about 2 trading years). `forecasts-api` combines two sources:

- **Table `Forecasts`:** closing prices of the days that already have a forecast (from 27/10/2026). When both sources have the same day, this one wins.
- **A JSON file on S3** (`serving/price-history/<asset>.json`): exported **once** with `scripts/export_price_history.py` from the clean data, converting units with the same function as Lambda `predict`. The export of 22/11/2026 holds 520 sessions per asset, up to the session of 20/11. Later sessions are already in table `Forecasts`, so the file does not need a daily export.

The Lambda reads S3 only when the table has fewer than N points, and keeps the file in memory for 30 minutes. If reading S3 fails, the API logs `PRICE_HISTORY_FILE_UNAVAILABLE` and still returns `200` with the part found in the table.

![S3 serving/price-history](/images/5-Workshop/5.3.1-s3-price-history.png)

*Folder `serving/price-history/` in bucket `cpaws-dev-lake`: 4 JSON files of about 24 KB each, exported on 22/11/2026.*

#### Deploy

```bash
cdk deploy CpawsServing --profile cpaws-dev
cdk deploy CpawsAuth --profile cpaws-dev
cdk deploy CpawsApi --profile cpaws-dev -c allowed_origin=https://<branch>.<app-id>.amplifyapp.com
```

The `CpawsAuth` outputs include `ForecastsUserPoolId` and `ForecastsUserPoolClientId`. The `CpawsApi` outputs include the API URL. The web app needs all three values at build time.

#### User accounts

Since 22/11/2026, User Pool `cpaws-dev-users` **allows self sign-up with email verification**, so graders can create their own accounts. This is a temporary setting with these limits:

- self sign-up closes by 10/12/2026 at the latest: a CDK PR sets `self_sign_up_enabled=False` again, the web app is rebuilt with `VITE_ALLOW_SIGNUP=false`, and accounts outside the allow list are disabled;
- each member invites at most 8 trial users through private messages; the link is never posted publicly;
- the chatbot limits of 20 questions per user per day and 100 per day for the whole system stay the same, so more users cannot push Bedrock cost past its cap;
- the web app has no feature to download price data.

`VITE_ALLOW_SIGNUP` only decides whether the web app shows the sign-up button, so it is not a security control: anyone with the App Client ID can still call the `SignUp` API while the User Pool allows it. What actually blocks sign-up is the User Pool setting.

The team can still create accounts by hand when needed, for example when a grader does not receive the verification email because the User Pool sends email with Cognito's default quota (`COGNITO_DEFAULT`):

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
| `VITE_ALLOW_SIGNUP` | Show the **Tạo tài khoản** (Create account) button (`true` in the submitted build during grading) |

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

A manual zip upload is not tied to any commit, so the team records the source of each deployment. Deployment 6 (22/11/2026) was built from branch `production`, tag `v1.0.0`; Deployment 7 (23/11/2026) is the running `v1.0.1`. The repository already contains `amplify.yml` so Amplify can later be connected to GitHub for automatic builds.

#### Interface of the submitted version (v1.0.1)

- **Public pages:** the landing page (`/`) and `/about`, with no prices and no calls to the forecast API. The overview, product pages, `/accuracy` and `/guide` (how to read forecasts) need sign-in; visitors who are not signed in are redirected to `/dang-nhap?next=…`.
- **Accounts:** the sign-up form asks for a full name; the sidebar shows the name and a sign-out button, and can be collapsed.
- **Product page chart:** ranges from 1 month to 2 years (loaded with `?history=`), with a Brush bar to zoom into one part.
- **Chatbot:** chat history is kept only for the browser session and is lost when the page is closed; the server still stores no conversations.
- **Display:** light and dark themes; the menu and pages work on phones.

Run the web app locally on sample data, without AWS:

```bash
cd web && npm ci && npm run dev   # http://localhost:5173
```

![API Gateway](/images/5-Workshop/5.4.3-api-gateway.png)

*Resource tree of API `cpaws-dev-forecasts-api`: `/health`, `/forecasts`, `/forecasts/{asset}`, `/accuracy/{asset}` and `/chat`; every resource also has `OPTIONS` for CORS. The API ID is hidden.*

![Cognito sign-up](/images/5-Workshop/5.4.3-cognito.png)

*The Sign-up section of User Pool `cpaws-dev-users`: Cognito sends the verification email; Self-registration is Enabled (temporarily, until 10/12/2026 at the latest).*

![Amplify deployments](/images/5-Workshop/5.4.3-amplify.png)

*Amplify app `cpaws-dev-web`, branch `dev`: 7 zip deployments; Deployment 7 (23/11/2026) is v1.0.1. The domain is hidden.*

![Landing page](/images/5-Workshop/5.4.3-login1.png)

*The public landing page with the sign-in and create-account buttons. The chart there is only an illustration, not real data.*

![Landing page (lower part)](/images/5-Workshop/5.4.3-login2.png)

*The lower part of the landing page: the 4 tracked products, the 4 steps that produce a forecast, the data sources (Yahoo Finance, U.S. EIA) and a note that forecasts are not investment advice.*
