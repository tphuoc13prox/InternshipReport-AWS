---
title: "Chatbot with Amazon Bedrock"
date: 2026-11-20
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---
#### Goals and constraints

The chatbot helps users read the forecasts in Vietnamese: questions about gold, silver, Brent and WTI at 1–7 session horizons, about concepts (the 80% range, the probability of a rise) and about related news. The chatbot **does not forecast on its own**: every number comes from a published record, so it matches the forecast page. The chatbot has three constraints:

1. **No investment advice.** It never recommends buying, selling, holding or allocating capital.
2. **Every number has a source.** No numbers outside tool results, no self-computed differences or conversions.
3. **Fit within API Gateway:** API Gateway cuts requests at 29 seconds, so the answer must finish before that.

#### The `POST /chat` flow

1. API Gateway checks the ID token with the Cognito authorizer. The user is identified by the `sub` claim, not by email.
2. Lambda `cpaws-dev-chat-api` (Python 3.12, 512 MB, 28-second timeout) validates the request: a question of 1–500 characters and at most 6 turns of history sent by the client. The server does not store conversations.
3. **Deadline check.** The Lambda computes a deadline from the time API Gateway received the request (29 seconds minus 4 seconds of margin). If there is not enough time left for one model call, it returns `504` before counting the question.
4. **Count the question** with one conditional DynamoDB write to table `chat-usage`. The limits are 20 questions per user per day and 100 per day for the whole system; beyond that it returns `429`. The table only stores counters, never questions.
5. **Call Bedrock Converse** with system prompt `chatbot-haiku-v2`, two tools, and at most 3 tool rounds.
6. **Return the result:**
   - the answer as plain text;
   - `citations` (the news items cited);
   - `forecasts_used` (asset, base date, model version, sample-data flag);
   - the remaining question quota and the disclaimer.
7. **Write one JSON log line:** branch, tools called, token counts, latency. The question text is not logged.

#### Model and invocation

| Item | Configuration |
|---|---|
| Model | Claude Haiku 4.5 through the **global** inference profile (cross-Region inference), called from `ap-southeast-1` |
| API | Converse (no streaming), `temperature` 0, `maxTokens` 1500, `toolChoice: auto` |
| IAM permissions | Only `bedrock:InvokeModel` on the exact ARNs of the inference profile and model; `dynamodb:Query` on the `forecasts` table |
| Not used | Knowledge Bases, embeddings, fine-tuning. News is read directly from labelled data |

With the global inference profile, Bedrock may process a request in another region. ADR-0004 states that the team does not guarantee every request is processed entirely in Singapore.

#### Router: a controlled negative result

The original design (ADR-0004) split questions into two branches. Simple questions (greetings, concepts) would go to **Amazon Nova 2 Lite** to save cost. Questions needing data would go to **Claude Haiku 4.5** with tools. The router chooses a branch by rules, without calling an LLM. It was tested as follows:

- 9 rules were **locked before** measuring.
- A held-out set of 30 questions was written by someone else; the rule author did not see it beforehand.
- Thresholds fixed in advance: accuracy ≥ 90%, and at most 1 question that needs Haiku wrongly sent to Nova.
- It was run exactly once.

Result: accuracy **26/30 (86.7%), below threshold**. No question needing Haiku went to Nova; all 4 errors were simple questions sent to Haiku, which only costs more. The team decided **not to put the router into `/chat`: every question goes to Haiku**, and not to tweak the rules and re-score on a set already seen. Cost is about 0.012 USD per question. With the cap of 100 questions per day, the chatbot costs at most about 1.2 USD per day.

#### Tools v2

| Tool | Source | Returns |
|---|---|---|
| `get_forecast(asset, horizon?)` | DynamoDB `cpaws-dev-forecasts`, `Query` with `Limit=1`. Formatted by the same function as the `forecasts-api` Lambda | q10, q50, q90 in %, probability of a rise, width of the 80% range, base date, `model_version`, sample-data flag. No price levels for gold/silver |
| `get_news(asset, from_date?, to_date?, min_impact?)` | Fed and EIA releases labelled by hand | Up to 10 items, each with title, team-written summary, link, source, date, impact 0–3 and direction |

Each simplification in tools v2 fixes an error seen in an earlier benchmark. For example, the probability of a rise is returned ready in % because the model once multiplied by 100 incorrectly; only q10/q90 are returned because the model once called the q05/q95 pair "the 80% range".

#### Calling Converse with tool use

```python
import boto3

bedrock = boto3.client("bedrock-runtime", region_name="ap-southeast-1")

TOOLS = [{"toolSpec": {
    "name": "get_forecast",
    "description": "Latest forecast (q10, q50, q90, probability of a rise) for one asset",
    "inputSchema": {"json": {
        "type": "object",
        "properties": {
            "asset": {"type": "string", "enum": ["gold", "silver", "brent", "wti"]},
            "horizon": {"type": "integer", "minimum": 1, "maximum": 7},
        },
        "required": ["asset"],
    }},
}}]  # get_news is declared the same way

def ask(question: str, model_id: str, system_prompt: str, run_tool) -> str:
    messages = [{"role": "user", "content": [{"text": question}]}]
    for _ in range(3):  # at most 3 tool rounds
        resp = bedrock.converse(
            modelId=model_id, system=[{"text": system_prompt}], messages=messages,
            toolConfig={"tools": TOOLS},
            inferenceConfig={"maxTokens": 1500, "temperature": 0},
        )
        msg = resp["output"]["message"]
        messages.append(msg)
        if resp["stopReason"] != "tool_use":
            return "".join(b.get("text", "") for b in msg["content"])
        # run every tool the model asked for and send the results back as toolResult
        results = [{"toolResult": {"toolUseId": b["toolUse"]["toolUseId"],
                                   "content": [{"json": run_tool(b["toolUse"]["name"], b["toolUse"]["input"])}]}}
                   for b in msg["content"] if "toolUse" in b]
        messages.append({"role": "user", "content": results})
    return "Mình chưa trả lời được câu này, bạn thử hỏi ngắn gọn hơn nhé."
```

#### Server-side safeguards

The model can ignore its prompt, so some rules are enforced again in code:

- `citations` only include news returned by `get_news` **in this turn** whose link appears in the answer. The web app only builds links from `citations` and never turns text in the answer into links.
- An answer containing a link that does not belong to a tool result is logged as `citation_link_mismatch` for review.
- Answers are displayed as plain text: no Markdown or HTML rendering.
- The disclaimer and `forecasts_used` are always in the response, even if the model forgets.
- Responses and error messages never contain the Account ID, ARNs, table names, model ID or system prompt.
- News content is **untrusted data**: the prompt tells the model to ignore any instruction found inside news.

The team **did not enable Bedrock Guardrails** within the project scope (decision of 11/11/2026). Investment-advice requests and prompt attacks are handled by prompt rules, the checks above and a benchmark that includes policy questions. Guardrails are listed as an improvement after the project.

#### Evaluation

- The benchmark calls the real Bedrock, with tools running on fixed data.
- Before a run, the questions, prompt, data and tool code are **locked with SHA-256**.
- The official run is done **once**; the prompt is never edited and re-scored on the same set.
- Prompt v4 failed its official run on 11/11/2026, so the Lambda keeps prompt v2. The whole benchmark run cost about 0.73 USD.

<!-- ẢNH: khung chat trên web: một câu hỏi dự báo + một câu hỏi tin tức có trích nguồn (CHE con số giá) -->
<!-- ẢNH: câu hỏi xin tư vấn mua/bán và câu trả lời từ chối -->
