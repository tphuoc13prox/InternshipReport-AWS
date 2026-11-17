---
title: "Blogs Posted"
date: 2025-09-09
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

At least 3 technical blogs published on the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/).

### [Blog 1 - AgentCore Memory: Why Does an Agent "Forget" What It Was Just Told?](3.1-Blog1/)

An original post inspired by the "AgentCore Memory" slide at FCAJ x FPTU: Cloud Mastery 1. It explains the two tiers of Amazon Bedrock AgentCore Memory (synchronous short-term events and asynchronously extracted long-term records) through five numbered data flows, shows why an agent cannot retrieve a preference the user just stated, and proposes reading both tiers (`ListEvents` + `RetrieveMemoryRecords`), direct ingestion with `BatchCreateMemoryRecords`/`IngestData`, and a troubleshooting checklist.

Published by Phan Tấn Phước on November 14, 2026: [original post on AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2337675363664089/)

### [Blog 2 - Tool Use with the Bedrock Converse API: The Model Called the Right Tool, So Why Is the Answer Still Wrong?](3.2-Blog2/)

An original post drawn from my experience building a chatbot on Amazon Bedrock, illustrated with a weather-chatbot example. It explains the five-step client-side tool use loop of the Converse API (`toolUse` → run the tool → `toolResult` → repeat until `end_turn`), the "brakes" a production loop needs (round limit, a deadline under the API Gateway timeout, handling every `stopReason`), structured tool errors, and why tool output should be designed for the model to read, ending with a checklist.

Published by Phan Tấn Phước on November 16, 2026: [original post on AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2339395856825373/)

### [Blog 3 - Prompt Caching for Chatbots with Tool Use: Place cachePoint in the Right Spot to Cut Up to ~90% of the Cost of Repeated Prompt Content on Amazon Bedrock](3.3-Blog3/)

An original post compiled from the AWS documentation, with illustrative numbers. It explains how prompt caching works in the Converse API (`cachePoint` in tools, system, and messages; cache write and cache read prices; TTL), estimates the savings for the repeated part of a tool use loop, and proposes how to arrange the prompt so the cache actually hits (static first, dynamic last; a stable tool list; a moving checkpoint at the end of the latest message), along with the pitfalls (minimum token threshold, hits not guaranteed) and how to measure the hit rate.

Published by Phan Tấn Phước on November 17, 2026: [original post on AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2340410270057265/)
