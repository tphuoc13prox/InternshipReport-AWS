---
title: "Bedrock chatbot"
date: 2025-09-09
weight: 2
chapter: false
pre: " <b> 5.5.2 </b> "
---

The chatbot helps users read CPAWS forecasts in Vietnamese: what the forecast for an asset and horizon says, what terms like "80% range" or "probability of rising" mean, and which Fed/EIA news came out recently. This is the part of CPAWS I was responsible for.

The chatbot **does not forecast anything itself**. Every number comes from a published forecast record through a tool, so the answer and the forecast page always show the same figures. It also never gives investment advice.

#### How one request is handled

1. API Gateway validates the Cognito ID token first. From then on the user is known only by the token's `sub`; the email is never used.
2. Lambda `cpaws-dev-chat-api` (Python 3.12, 512 MB, 28 s) checks the body. The `question` must be 1–500 characters; `context.asset` (the page the user is on) is optional; `history` may hold up to 6 turns. Conversations are kept in the browser, not on the server. Invalid input returns `400` and does not count against the quota.
3. **Deadline.** API Gateway gives up after 29 s, counted from `requestTimeEpoch`, so the Lambda counts from the same point and keeps 4 s in reserve. Authorizer time and cold start are therefore already inside the budget. When the remaining time is too short for another model call, the Lambda answers `504` itself rather than letting the gateway cut the connection.
4. **Quota.** Each user may ask 20 questions a day and the whole system 100, with the counters starting again at midnight Vietnam time. Both counters in `cpaws-dev-chat-usage` are increased in a single `TransactWriteItems`, each with a condition; if one condition fails the whole transaction is cancelled, no counter moves, and the user receives `429 rate_limited`.
5. **Model call.** Bedrock Converse with Claude Haiku 4.5, system prompt `chatbot-haiku-v2`, `temperature` 0, `max_tokens` 1500 and at most three rounds of tool calls.
6. **Response.** `answer` (plain text), `forecasts_used`, `citations`, `quota`, `disclaimer` and a few flags. One JSON log line records the tools called, tokens and duration, but not the question text.

The answer is returned in one piece, not streamed. That keeps the Cognito authorizer and the Python Lambda as they are, and the measured response time is far below the 29-second limit.

#### The two tools

| Tool | What it returns | Why it is shaped this way |
|---|---|---|
| `get_forecast(asset, horizon?)` | Latest record from `cpaws-dev-forecasts`, formatted by the same function as the forecast API, then reduced: only q10, q50, q90 in %, the probability of rising in %, and the width of the 80% range; oil prices in USD/bbl; **no price level for gold and silver** | In the first benchmark the model mixed up quantile pairs, multiplied percentages itself and quoted gold prices. Giving it fewer, ready-made numbers removed those errors |
| `get_news(asset, from_date?, to_date?, min_impact?)` | Labelled Fed/EIA news with title, summary, link, source, date, impact score 0–3 and direction; default 7 days, `min_impact` 2, at most 10 items | The chatbot must cite a link for every news item, so items without a valid link are dropped |

News labels are made by hand: 2–3 Fed/EIA releases per week, packaged as JSON with the Lambda (ADR-0013). An automatic labelling step with Nova 2 Lite was tried first but failed its checks (direction 52.4% against a target of 80%), so the team chose manual labels until the end of the project. Changing the news set means redeploying `CpawsApi`. The latest set went live with release v1.0.0 on 22/11: the FOMC minutes of 27–28/10 (for gold and silver) and the EIA weekly petroleum report of 18/11 (for Brent and WTI).

#### Bedrock access and IAM

The model is called through the global cross-Region inference profile `global.anthropic.claude-haiku-4-5-20251001-v1:0` from `ap-southeast-1` (see [5.2](../../5.2-prerequiste/)). Because the profile is global, the team does not promise that every request is processed inside Singapore.

The Lambda role has exactly what it needs:

+ `bedrock:InvokeModel` on the three ARNs of the profile and the model, with no streaming permission;
+ `dynamodb:Query` on the forecast and news tables;
+ `dynamodb:UpdateItem` and `GetItem` on `cpaws-dev-chat-usage`.

#### Protections that do not depend on the model

The model does not always obey its prompt, so the rules that matter most are checked a second time in code:

+ An item reaches `citations` only if `get_news` returned it during this request and its link is present in the answer. Clickable links on the web come from `citations` alone; text in the answer is never turned into a link.
+ Any answer containing a URL that does not belong to that request's news is logged as `citation_link_mismatch` (request ID, prompt version and the URLs, no question or answer).
+ `disclaimer` and `forecasts_used` are always in the response, so the web shows the disclaimer and the "sample data" label even if the answer forgets them.
+ `history` from the client is never a source of numbers; to quote a figure the model must call `get_forecast` again.
+ Nothing internal leaks into a response: no account ID, ARN, table name, model ID, system prompt or stack trace, whether the request succeeds or fails.

Bedrock Guardrails were planned as a third layer, but the guardrail is still `DRAFT` and not attached to the Lambda.

#### Deploy and test

The version on `/chat` today is release **v1.0.0** (tag `v1.0.0`, commit `ba3d9f4`), deployed with `CpawsApi` on 22/11/2026, still with prompt `chatbot-haiku-v2`. The next patch, v1.0.1, adds a second `system` block with the current date and time in Vietnam so the model stops confusing "today" with the forecast's base date; the locked v2 prompt itself does not change. It is approved but not yet merged or deployed.

The chat code, the two v2 prompt files and the news JSON are copied into the Lambda asset at `cdk synth`. Any change to them is deployed with `CpawsApi`:

```bash
cdk diff CpawsApi -c allowed_origin=ORIGIN_WEB --profile cpaws-dev
cdk deploy CpawsApi -c allowed_origin=ORIGIN_WEB --profile cpaws-dev
```

Test with an ID token (pasted with `read -rs ID_TOKEN && export ID_TOKEN`):

```bash
curl -sS -X POST "$API_URL/chat" \
  -H "Authorization: $ID_TOKEN" -H "Content-Type: application/json" \
  -d '{"question": "Dự báo bạc 5 phiên tới thế nào?", "context": {"asset": "silver", "page": "product"}}'
```

| Status | `error` | Meaning |
|---|---|---|
| 400 | `invalid_body`, `empty_question`, `question_too_long`, `invalid_context`, `invalid_history` | Bad request, not counted |
| 429 | `rate_limited` | Daily quota used up (`reset_at`, `Retry-After`) |
| 503 | `model_unavailable` | Bedrock still failing after one retry |
| 504 | `timeout` | Not enough time left for the next model call |
| 500 | `internal_error` | Anything else; details only in CloudWatch with the `request_id` |

To find answers with unexpected links, run this in CloudWatch Logs Insights on the chat log group:

```
fields @timestamp, request_id, prompt_version
| filter event = "citation_link_mismatch"
| sort @timestamp desc
```

<!-- IMAGE: web chat panel on the silver page, answer with 80% range, probability of rising and disclaimer -->
[ẢNH: chat panel on the silver page with the disclaimer]

#### How the chatbot was evaluated

Every prompt change was measured with a benchmark that calls the real Bedrock model, with tools reading a fixed fixture. Questions, prompt, fixture and tool code were locked with SHA-256, thresholds were set before the run, and each official set was run exactly once. I graded every answer (with Claude suggesting a score and a reason), and Khải graded a sample independently without using a model.

| Run | Result | Threshold | Decision |
|---|---:|---:|---|
| Prompt v2, set B (30 questions needing tools), 30/10 | 14/30 | ≥ 25 | Not passed; kept on `/chat` because no later prompt met its conditions |
| Rule-based router, held-out set, 04/11 | 26/30 | ≥ 27 (90%) | Not used; every question goes to Haiku |
| Prompt v4, new set B, 11/11 | 17/30 | ≥ 25 | Not passed; v2 kept |
| Prompt v4, new set A (8 simple questions), 11/11 | 5/8 | ≥ 7 | Not passed |

Across all official runs there were **0 policy violations** and the model never followed instructions injected into news text. The remaining failures are mostly about reading numbers (dropping a minus sign, comparing negative numbers the wrong way, calling q90 a "maximum") and missing required parts. On set B with v2, the median response time was 4,023 ms and p90 5,733 ms.

**Cost.** About 0.012 USD per question with Haiku. With the 100-per-day cap, the chatbot can spend at most about 1.2 USD a day, or about 36 USD a month in the worst case, below the project's 50 USD monthly budget.
