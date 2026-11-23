---
title: "Cognito and REST API"
date: 2025-09-09
weight: 1
chapter: false
pre: " <b> 5.5.1 </b> "
---

#### Step 1 – User pool (`CpawsAuth`)

`CpawsAuth` creates the user pool `cpaws-dev-users` and one app client for the single-page app:

+ email verified automatically with a code sent to that address;
+ passwords of at least 8 characters with upper case, lower case, digits and symbols;
+ app client `cpaws-dev-app` with no client secret (a browser cannot keep one), SRP and user-password flows, no OAuth, and `prevent_user_existence_errors` so the sign-in form does not reveal which emails exist;
+ `RETAIN`, so deleting the stack does not delete users.

```bash
cdk diff CpawsAuth --profile cpaws-dev
cdk deploy CpawsAuth --profile cpaws-dev
```

The outputs `ForecastsUserPoolId` and `ForecastsUserPoolClientId` go into the web build in [5.6](../../5.6-web-amplify/). Any browser app has to expose these two values, so they are not treated as secrets.

**Self sign-up.** Normally accounts are created by the team. For the grading period, ADR-0014 opens self sign-up with email verification. The change was deployed to `CpawsAuth` on 22/11/2026; CloudFormation updated three attributes of the existing pool in place, so no users were lost. The web was built with `VITE_ALLOW_SIGNUP=true` at the same time. According to the plan, sign-up is closed again within one day after the mentor confirms grading is finished, and no later than 10/12/2026: first in `CpawsAuth`, then in the web build, then the self-created users are disabled ([5.10](../../5.10-cleanup/)).

To create a test user yourself (use your own address, never commit it):

```bash
aws cognito-idp admin-create-user --user-pool-id <USER_POOL_ID> \
  --username <TEST_EMAIL> --profile cpaws-dev
```

<!-- IMAGE: Cognito console, user pool cpaws-dev-users, Sign-in experience tab (no user emails visible) -->
[ẢNH: Cognito console – user pool `cpaws-dev-users`]

#### Step 2 – REST API (`CpawsApi`)

The API is one REST API with stage `dev`. Settings worth knowing before you deploy:

+ **Cognito authorizer** on every route except `/health`. The API trusts only an ID token from `cpaws-dev-users`, sent in the `Authorization` header.
+ **Stage throttling** of 10 requests per second with a burst of 20. This is the first cost guard for the chatbot; the per-user quota in [5.5.2](../5.5.2-bedrock-chatbot/) is the second.
+ **CORS for exactly one origin**, given with `-c allowed_origin`. That value is set in three places: the `OPTIONS` preflight, the response headers of the Lambda functions, and the gateway error responses, which lets the browser read the status code of a 401, 429 or 504.

Deploy with the origin that will call the API. While developing locally that is `http://localhost:5173`; after [5.6](../../5.6-web-amplify/) it is the Amplify domain:

```bash
cdk diff CpawsApi -c allowed_origin=ORIGIN_WEB --profile cpaws-dev
cdk deploy CpawsApi -c allowed_origin=ORIGIN_WEB --profile cpaws-dev
```

{{% notice warning %}}
CDK does not remember `-c` values between deploys. Forget `allowed_origin` once and the origin silently returns to `http://localhost:5173`, which breaks the web on Amplify. Include it every time.
{{% /notice %}}

The output `ForecastsApiUrl` is the base URL for the web and for the tests below.

#### Step 3 – Test the routes

```bash
curl -sS "$API_URL/health"
curl -sS "$API_URL/forecasts/gold"                              # expect 401
curl -sS "$API_URL/forecasts/gold" -H "Authorization: $ID_TOKEN" # expect 200
```

To keep the token out of your shell history, paste it with `read -rs ID_TOKEN && export ID_TOKEN`.

A forecast record contains the seven quantiles q05…q95 of the log return for each horizon, the probability of rising, the base date, `model_version` (`baseline-q252`) and flags for sample or stale data. The web turns q10–q90 into the nominal 80% forecast range.

<!-- IMAGE: terminal, /health 200 then /forecasts/gold 401 without token and 200 with token (token hidden) -->
[ẢNH: terminal – `/health` 200, `/forecasts/gold` 401 then 200]
