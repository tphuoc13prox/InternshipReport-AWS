---
title: "Demo"
date: 2025-09-09
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Links

| Item | Link |
|---|---|
| Production web (Amplify) | [CẦN BỔ SUNG: link Amplify của nhóm] |
| Team demo video | [CẦN BỔ SUNG: link video của nhóm] |
| Source code | See [5.2](../5.2-prerequiste/) |

Apart from the public landing page, the web is behind sign-in. During grading you can create your own account with **Tạo tài khoản** (email verification required). No login details are published on this page.

The team video covers the web, the daily pipeline and operations. The chatbot is not part of the video; it is shown with the screenshots below.

#### Suggested walkthrough

1. **Create an account or sign in** from the landing page on the Amplify domain. Every page after the landing page needs an ID token from Cognito.
2. **Home page.** Four asset cards: gold, silver, Brent, WTI.
3. **Asset page, for example silver.** Seven horizon cards (1–7 sessions). Each card shows the nominal 80% forecast range, the probability of rising and the 60/40 label. Below them is the 2-year price chart. Switch to advanced mode for the q10/q50/q90 table and the price range per horizon.
4. **Money conversion.** Enter an amount in VND and see the range of values it maps to for the selected horizon.
5. **"Độ tin cậy" block.** It shows "Đang thu thập (n/20)" until 20 base dates are scored ([5.4.3](../5.4-data-pipeline/5.4.3-rolling-accuracy/)).
6. **Chat panel.** Ask about the asset you are viewing. The answer quotes the same figures as the cards, names the base date and horizon, and ends with the disclaimer.
7. **Behind the scenes.** In CloudWatch, the 19:15 predict log and the six alarms in OK.

<!-- IMAGE: home page with four asset cards -->
[ẢNH: home page – four asset cards]

<!-- IMAGE: asset page in advanced mode with the q10/q50/q90 table -->
[ẢNH: asset page – advanced mode]

<!-- IMAGE: "Độ tin cậy" block showing Đang thu thập (n/20) -->
[ẢNH: "Độ tin cậy" block]

#### Chatbot examples

Questions that show what the chatbot is built for:

| Question | What to look for |
|---|---|
| "Dự báo bạc 5 phiên tới thế nào?" | 80% range, median and probability of rising from `get_forecast`; base date; the statement that `baseline-q252` is a 252-session statistical baseline; disclaimer |
| "Tuần này có tin gì ảnh hưởng tới dầu WTI không?" | Fed/EIA items with source, date and link, shown separately from the forecast |
| "Có nên mua vàng bây giờ không?" | A short refusal: "Mình không đưa ra lời khuyên đầu tư." |

<!-- IMAGE: chat panel, forecast answer for silver with disclaimer -->
[ẢNH: chat – forecast answer for silver]

<!-- IMAGE: chat panel, refusal of an investment-advice question -->
[ẢNH: chat – investment advice refused]
