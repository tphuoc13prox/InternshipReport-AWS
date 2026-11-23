---
title: "Web on Amplify Hosting"
date: 2025-09-09
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### What the web is

The web lives in `web/`: React + TypeScript + Vite + Tailwind, TanStack Query for API calls, Recharts for charts, and the Amplify UI `Authenticator` on the existing Cognito user pool (no Amplify backend). Since v1.0.0 there is a public landing page; everything else is behind sign-in. The web also has a dark mode. Each API call carries the ID token in `Authorization`, and a 401 from the API sends the user back to the sign-in screen.

| Page | Content |
|---|---|
| Landing (public) | What CPAWS is, sign-in and sign-up |
| Home | Four asset cards |
| Asset (gold, silver, Brent, WTI) | Seven horizon cards, money conversion block, fan chart, 2-year price chart, advanced table, "Độ tin cậy" block, chat panel |
| "Độ chính xác" | Accuracy block for all four assets and how to read the metrics |
| "Giới thiệu" | How to read a forecast, the method, data sources |

Display rules the team agreed on:

+ **Always a range with a probability**, never a single price. The model forecasts log returns; the web turns them into a price range using the latest close.
+ **60/40 rule for direction labels.** Rounded probability of rising ≥ 60% is "Nghiêng tăng", ≤ 40% is "Nghiêng giảm", anything in between is "Chưa rõ xu hướng". The label and the number use the same rounded value, so they never contradict each other.
+ **VND conversion** with the exchange rate in the snapshot: gold and silver in VND per lượng (1 lượng = 37.5 g), oil in VND per barrel. The gold label says it is a converted world price, not the domestic bar price.
+ **Gold and silver** are converted from GLD and SLV, and the "Giới thiệu" page says so. Their prices are shown only after sign-in, with the source line, and with no download feature.
+ Every page says the forecast is not investment advice.
+ The 2-year price-history chart was added in v1.0.0. The price series is served by forecasts-api from a file in S3, so the browser never calls the price source directly.

<!-- IMAGE: asset page for gold, seven horizon cards with 80% range, probability of rising and 60/40 label -->
[ẢNH: gold asset page – seven horizon cards]

#### Step 1 – Run locally

Create `web/.env.local` with the region, the user pool ID and app client ID from `CpawsAuth`, the API URL from `CpawsApi`, and the feature flags:

| Flag | Effect |
|---|---|
| `VITE_AUTH_ENABLED` | Sign-in required |
| `VITE_SHOW_METAL_PRICES` | Show gold and silver prices (off: only % change and probability) |
| `VITE_ACCURACY_API` | Call `/accuracy/{asset}` and show the "Độ tin cậy" block |
| `VITE_ALLOW_SIGNUP` | Show the sign-up and email-code screens of the `Authenticator` (default off) |

```bash
cd web
npm ci
npm run dev
```

The dev server runs on `http://localhost:5173`, which is the default `allowed_origin` of `CpawsApi`. The same checks as CI can be run with `npm run typecheck`, `npm test` and `npm run build`.

`VITE_ALLOW_SIGNUP` only hides or shows screens. It does not block anyone, because the app client ID is public; the real control is the user pool setting in [5.5.1](../5.5-api-and-chatbot/5.5.1-cognito-and-api/).

#### Step 2 – Create the Amplify app

The team created the app by hand on 04/11 so it could be ready for the rehearsal, and left the GitHub connection for later, because connecting GitHub creates a new app and a new domain.

1. Open **AWS Amplify** → **Create new app** → **Deploy without Git**.
2. App name `cpaws-dev-web`, branch `dev`.
3. Add a rewrite so every SPA route returns `index.html` (source `</^[^.]+$|\.(?!(css|gif|ico|jpg|js|png|txt|svg|woff|woff2|ttf|map|json)$)([^.]+$)/>`, target `/index.html`, type `200 (Rewrite)`).

<!-- IMAGE: Amplify console, app cpaws-dev-web, branch dev, Deploy updates button -->
[ẢNH: Amplify console – app `cpaws-dev-web`, branch `dev`]

#### Step 3 – Build and upload

Vite bakes the `VITE_*` values into the bundle **when it builds**. To change a flag, rebuild and upload; the API stays as it is.

```bash
cd web
npm run build
```

Zip the contents of `web/dist`, then in the console open `cpaws-dev-web` → branch `dev` → **Deploy updates** and drop the zip. Open the domain, sign in, and check the forecast page and the chatbot.

Because a zip upload is not linked to a commit, the team writes down which commit and flags each deployment used:

| Date | Commit | `VITE_SHOW_METAL_PRICES` | `VITE_ACCURACY_API` |
|---|---|---|---|
| 04/11 | `047189f` | `false` | `false` |
| 07/11 | `c94849a` | `true` | `false` |
| 08/11 | `2e7d3fa` | `true` | `false` |
| 13/11 | `26399d0` | `true` | `true` |
| 22/11 | release `v1.0.0` (`ba3d9f4`), also `VITE_ALLOW_SIGNUP=true` | `true` | `true` |

#### Step 4 – Point the API at the Amplify domain

```bash
cdk deploy CpawsApi -c allowed_origin=https://dev.<APP_ID>.amplifyapp.com --profile cpaws-dev
```

Then check in the browser's developer tools that the preflight returns the Amplify origin and there are no CORS errors. Because the API accepts only one origin, the web on Amplify and the web on localhost cannot call it at the same time. To go back to local development, deploy again with `http://localhost:5173`; it takes about 3 minutes.

**Production URL:** [CẦN BỔ SUNG: link Amplify của nhóm]
