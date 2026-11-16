---
title: "Blog 2"
date: 2026-11-16
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Tool Use with the Bedrock Converse API: The Model Called the Right Tool, So Why Is the Answer Still Wrong?

by Phan Tấn Phước | November 16, 2026 | published in [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Original post on Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2339395856825373/)

*An original post based on my experience building a chatbot on Amazon Bedrock; the example in the post (a weather chatbot) is illustrative. The post was written and published in Vietnamese; this page is an English translation that stays faithful to the published version.*

Recently I have been building a chatbot on Amazon Bedrock, using the Converse API so that the model can call tools to fetch real data. One situation surprised me: the model called the right tool, the tool returned the right data, and yet the final answer was still wrong. In this post I describe how the tool use loop works and what I learned about designing tools. To keep things concrete, I use a weather chatbot as the example; the numbered diagram is shown in Figure 1.

### 1. The model does not call the tool itself

The first thing to understand: on Bedrock, the model does not run the tool directly. The model only says "I need to call this tool with this input", and your code runs the tool and sends the result back. A single turn goes through five steps:

- **(1)** The application calls Converse, sending the messages together with a `toolConfig`. Each tool has a `toolSpec` with a `name`, a `description`, and an `inputSchema` (JSON Schema).
- **(2)** The model returns `stopReason = "tool_use"`. The message contains a `toolUse` block with a `toolUseId`, a `name`, and an `input`, for example `get_weather_forecast` with `{"city": "Ho Chi Minh", "days": 3}`.
- **(3)** Your code runs the corresponding function, for example a Lambda function that reads data from DynamoDB.
- **(4)** You send back a message with the role `"user"` containing a `toolResult` block with that same `toolUseId`, the `content` (JSON or text), and a `status`.
- **(5)** You call Converse again. The model may request another tool (back to step 2) or finish its answer with `stopReason = "end_turn"`.

![Tool use loop with the Bedrock Converse API](/images/3-Blog/blog2_figure1.png)

*Figure 1. The tool use loop illustrated with a weather chatbot (five numbered steps), drawn by me based on the AWS documentation*

The `toolUseId` is what lets the model match each result to the right request. When a user asks something like "What will the weather be like in Saigon and Hanoi tomorrow?", the model may request several tools in the same turn; in that case, run all of them and put all the `toolResult` blocks into a single message.

### 2. The loop needs "brakes"

Examples in the documentation usually call a tool only once. When you put this into real use, you should add three things:

- **A limit on the number of rounds,** for example at most three tool rounds. If the model still requests a tool after the limit, return a fixed message ("I can't answer this question yet, please try asking more specifically") rather than a half-finished preamble such as "Let me look that up...".
- **A deadline for the whole turn.** If the chatbot sits behind API Gateway (REST API), the default maximum integration timeout is 29 seconds (it can be raised for Regional APIs). Before each model call, check whether the remaining time is enough for one call (the client's connect timeout plus read timeout). If not, have Lambda return a properly formatted timeout error itself instead of letting API Gateway cut the request off.
- **Handle every `stopReason`,** not just `"tool_use"` and `"end_turn"`. Converse also returns `max_tokens`, `stop_sequence`, `guardrail_intervened`, `content_filtered`, `malformed_model_output`, `malformed_tool_use`, and `model_context_window_exceeded`. For example, on `max_tokens` mark the answer as truncated, and on `guardrail_intervened` return a fixed refusal message.

Pseudocode for each turn:

```python
messages = history + [question]
for round in 0..MAX_ROUNDS:
    if time_remaining() < time_for_one_call: return timeout error
    resp = Converse(modelId, system, messages, toolConfig)
    messages.append(resp.output.message)
    if resp.stopReason != "tool_use": break
    if round == MAX_ROUNDS: return fixed message; stop
    results = []
    for tu in toolUse blocks:
        out = run_tool(tu.name, tu.input)
        results.append(toolResult(tu.toolUseId, json=out, status="error" if failed else "success"))
    messages.append({role: "user", content: results})
```

One thing to know about cost: Converse does not keep the conversation for you, so every tool round is another model call that must resend all the messages, including earlier tool results. Input tokens therefore grow quickly with the number of rounds and are usually many times larger than output tokens. Limiting the number of rounds also limits cost.

### 3. When a tool fails, report the error; do not let the model guess

- Return tool errors in a structured form, for example `{"error": "not_found", "message": "No forecast data for this city yet"}`, and set `status = "error"` in the `toolResult`. According to the API reference, the `status` field is only supported by Amazon Nova and Anthropic Claude 3 and 4 models, so for other models put the error information directly in the `content`.
- The `inputSchema` should use `enum`, `minimum`, and `maximum`, but the tool code must still validate the input. The schema helps the model send correct input; it does not replace server-side validation.
- One failing data source should not break the whole answer. For example, if the forecast tool works but the weather alerts tool (`get_weather_alerts`) fails, return an error for that tool only so the model can still answer the rest, instead of the whole request becoming a 500.
- Tool results that come from external sources (news, web pages, email) are untrusted data and may contain prompt injection. State this clearly in the tool's `description` and in the system prompt.

### 4. Calling the right tool is not enough: design the output for the model to read

This was my biggest lesson. Suppose the forecast tool returns nearly raw data: the temperature at seven percentiles from p05 to p95. A user asks, "What range is tomorrow's temperature most likely to fall in?" The model misreads the data, takes p25 and p75 (a 50% range), and calls it "the 80% range". Or it subtracts two numbers on its own to compute a difference and returns a figure that does not exist in the data.

I ran into exactly this kind of percentile misreading. The worrying part is that every number the model quoted really was in the tool result, so an automated check of the form "is each number in the answer present in the data?" could not catch it; only manual review found it.

The fix follows one principle: **anything the model might compute wrongly, compute it in code beforehand.**

- Return only the fields that are really needed, with names that state their meaning: `temp_low_80`, `temp_median`, `temp_high_80` instead of seven bare percentiles.
- Precompute the numbers people often ask about: `rain_prob_pct` (probability of rain in %), `width_80` (the width of the 80% range).
- Convert units beforehand and include the unit in the field name or the description.
- Have the tool description explain what each field means.

Fixing the output does not make the model stop making mistakes: once one error disappears, another may surface, such as comparing two negative numbers incorrectly. Tool use solves the problem of getting the right data; reading that data correctly requires continued work on both the tool output and the prompt, plus a test question set to re-measure after each change.

### 5. Checklist for tool use

- Does the tool description clearly state what each field means and its unit?
- Does the `inputSchema` use `enum`, `minimum`, and `maximum`? Does the tool code validate the input again?
- Is there a limit on tool rounds and a deadline for the whole turn?
- Is every `stopReason` value handled?
- Are tool errors returned in a structured form, with `status "error"` (if the model supports it)?
- `toolChoice`: `"auto"` is the default and is supported by every model that supports tool use; `"any"` and `"tool"` are supported only by some models, and sending an unsupported one results in a `ValidationException`.
- Do you log the tools called, tool errors, and tokens for each turn so you can review them?

### Conclusion

With the Converse API, tool use is a loop controlled by your own code: the model proposes, the code executes, and the result goes back to the model. The hard part is not calling the API but two things: putting "brakes" on the loop (rounds, time, `stopReason`) and designing the tool output so that the model is unlikely to misread it. The tool output is, in effect, part of the prompt too.

*Note: the list of models that support each feature (toolChoice, status, streaming with tools) changes over time, so check the latest documentation before applying this.*

### References

- [Tool use (function calling)](https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use.html)
- [Client-side tool use](https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use-client-side.html)
- [Use the Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html)
- [Converse API reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html)
- [ToolResultBlock](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_ToolResultBlock.html)
- [API Gateway REST API quotas](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-execution-service-limits-table.html)

#AWS #AmazonBedrock #ConverseAPI #ToolUse #FunctionCalling #AIAgent #AWSStudyGroup #FCAJ
