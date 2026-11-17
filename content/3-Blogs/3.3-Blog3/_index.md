---
title: "Blog 3"
date: 2026-11-17
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Prompt Caching for Chatbots with Tool Use: Place cachePoint in the Right Spot to Cut Up to ~90% of the Cost of Repeated Prompt Content on Amazon Bedrock

by Phan Tấn Phước | November 17, 2026 | published in [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Original post on Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2340410270057265/)

*An original post compiled from the AWS documentation; the numbers in the post are illustrative. The post was written and published in Vietnamese; this page is an English translation that stays faithful to the published version.*

In my previous post about tool use with the Converse API, I mentioned that Converse does not keep the conversation for you, so every tool round is another model call that must resend all the tool definitions, the system prompt, and the history. A question that needs two tool rounds means three calls, and that "identical" content is billed three times. Prompt caching is a way to pay much less for that repeated content while also reducing latency. In this post I summarize the AWS documentation and explain how to arrange the prompt so that the cache actually "hits". The numbers in the post are illustrative; the diagram is shown in Figure 1.

### 1. How does prompt caching work?

Bedrock caches by prompt prefix. You set cache checkpoints with a `cachePoint` block; the content before the checkpoint is stored, and a later request with the same prefix reads it from the cache instead of processing it again.

- There are two types: implicit (Bedrock tries to reuse the prefix on its own, with no configuration, but only on a best-effort basis) and explicit (you place `cachePoint` yourself). For Claude on Bedrock you must use explicit caching: requests without cache controls are not cached.
- In Converse, `cachePoint` can be placed in three places: `tools`, `system`, and `messages`. They are processed in the order tools → system → messages and are chained: changing the tools also invalidates the cache for the system and messages that follow.
- Claude allows at most four cache checkpoints per request.
- The cache has a TTL, five minutes by default, which is reset on every cache hit. Some models support a one-hour TTL (`"ttl": "1h"`).

The syntax is very short; you just add one block after the content you want to cache:

```json
{"cachePoint": {"type": "default"}}
```

![Prompt caching in the tool use loop with the Converse API](/images/3-Blog/blog3_figure1.png)

*Figure 1. Where to place cachePoint in the tool use loop, with an illustrative cost comparison, drawn by me based on the AWS documentation*

### 2. Where do the savings come from?

Input tokens now have three price levels: tokens written to the cache, tokens read from the cache, and regular tokens. According to the Bedrock pricing page, for many Claude models a five-minute cache write costs 1.25 times the regular input price, a one-hour cache write costs 2 times, and a cache read costs only 0.1 times (some models are even lower). The output token price does not change.

Illustrative example: a chatbot whose tool definitions plus system prompt are 5,000 tokens long, with an average of three model calls per question (two tool rounds). Counting only the 5,000 repeated tokens, in "units = the price of one regular input token":

- No cache: 3 × 5,000 = 15,000 units.
- With caching, cache empty (first question): write 5,000 × 1.25 + read 2 × 5,000 × 0.1 = 7,250 units, about 52% less.
- With caching, cache "warm" (another request arrived within the previous five minutes): read 3 × 5,000 × 0.1 = 1,500 units, 90% less.

The more tool rounds the chatbot makes, the longer the system prompt, and the more steadily users ask questions, the bigger the gain. Besides cost, the documentation lists two more benefits: lower latency, because the model does not have to reprocess the cached part, and tokens read from the cache are not deducted from your rate limit.

### 3. The idea: arrange the prompt so the cache hits

Turning on `cachePoint` is easy; making it hit is the hard part. Because the cache matches on the exact prefix, a single changed character near the start makes everything after it miss. Here are the principles I took away:

- **Static first, dynamic last.** The ideal order is: tool definitions → fixed system prompt → conversation history → new question.
- **Move dynamic content out of the system prompt.** A very common mistake is inserting "Today is 17/11/2026, 10:32" or the user's name at the start of the system prompt: it differs on every request, so the cache never hits. Move this information into the user message, after the cache checkpoint.
- **Keep the tool list fixed and in a stable order.** If your code builds `toolConfig` from a dict or set whose order is not stable, or if each user group gets a different set of tools, the prefix differs and the cache misses. Sorting the tools by name before sending them is enough.
- **Use a "moving" cache checkpoint in the tool loop.** Besides the checkpoints after tools and system, add one more `cachePoint` at the end of the latest message (the message containing the `toolResult`). The next round can then read the history of the previous round as well, not just the tools and system part. Note that the documentation says Claude only looks back about 20 content blocks before each checkpoint, so long conversations should have several checkpoints.
- **Split checkpoints by how often content changes.** For example, one checkpoint after the tools and one after the system prompt: when you edit the system prompt, the tools are still read from the cache.

Newer Claude models also offer "automatic caching": add a `cache_control` field at the request level (through `additionalModelRequestFields` in Converse), and Claude places the checkpoint on the last block and moves it forward as the conversation grows. This is simpler, but you still need to follow the "static first, dynamic last" principle.

### 4. Pitfalls to know

- **Minimum threshold:** the prefix before each checkpoint must reach the model's minimum token count, currently between 512 and 4,096 tokens depending on the model (for example, Claude Haiku 4.5 needs 4,096 and Claude Sonnet 4.6 needs 1,024). Below the threshold the request still runs normally but NOTHING is cached, and no error is reported. The threshold is counted cumulatively across tools + system + messages.
- **Hits are not guaranteed:** the cache lives on the server that processed the request. A request that lands on a different server (for example, in another Region when using cross-Region inference), or identical requests sent before the first one has started responding, write a new entry instead of reading one. The documentation also notes that at times of high demand, cross-Region inference can increase cache writes.
- **Writes cost more than reads:** if traffic is sparse (users ask more than five minutes apart), the cache keeps expiring and you pay the 1.25x write price without reading it back. In that case, consider the one-hour TTL (writes at 2x the price) or no caching at all.
- Caching works only with on-demand inference; batch inference is not supported.
- Setting `ttl` on a model that supports only the five-minute TTL can result in a `ValidationException`.

### 5. Measure to know whether it works

The Converse response includes `cacheReadInputTokens` and `cacheWriteInputTokens`. Note that when caching is enabled, `inputTokens` covers only the NON-cached part, so:

```text
total input tokens = inputTokens + cacheReadInputTokens + cacheWriteInputTokens
```

I recommend logging these three numbers for every call and tracking the cache read ratio = cacheRead / total input tokens. If `cacheWriteInputTokens` is always 0, the prefix most likely does not reach the threshold; if every call writes but never reads, some dynamic content is sitting before the cache checkpoint.

### Checklist

- Does your model support prompt caching, what is its minimum token threshold, and which TTL does it support?
- Are the tools and the system prompt fixed and in the same order across requests?
- Have you moved the date and time, the user's name, and session data out of the part before the cache checkpoint?
- Is there a cache checkpoint at the end of the latest message in the tool loop?
- Is traffic dense enough for the five-minute cache to stay "warm"? If not, is the one-hour TTL worth it?
- Are you logging `cacheReadInputTokens` / `cacheWriteInputTokens` to measure the hit rate?

### Conclusion

For a chatbot with tool use, the repeated part of the prompt is usually much larger than the user's new question. Prompt caching turns that repeated part into "cache read" tokens that cost only about a tenth of the price, but only if you arrange the prompt as static first, dynamic last, and verify it with the cache figures in the response. It is one of the optimizations with the best "effort spent / money saved" ratio that I know of on Bedrock.

*Note: the list of supported models, the token thresholds, the TTLs, and cache prices change fairly often, so check the latest documentation and pricing before applying this.*

### References

- [Prompt caching for faster model inference](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html)
- [CachePointBlock (API reference)](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_CachePointBlock.html)
- [Use the Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html)
- [Tool use (function calling)](https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use.html)
- [Amazon Bedrock pricing](https://aws.amazon.com/bedrock/pricing/)

#AWS #AmazonBedrock #PromptCaching #ConverseAPI #ToolUse #CostOptimization #GenAI #AWSStudyGroup #FCAJ
