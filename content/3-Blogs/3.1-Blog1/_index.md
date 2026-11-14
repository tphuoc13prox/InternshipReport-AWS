---
title: "Blog 1"
date: 2026-11-14
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# AgentCore Memory: Why Does an Agent "Forget" What It Was Just Told?

by Phan Tấn Phước | November 14, 2026 | published in [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Original post on Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2337675363664089/)

*An original post, inspired by the "AgentCore Memory" slide at the FCAJ x FPTU: Cloud Mastery 1 event (November 14, 2026). The post was written and published in Vietnamese; this page is an English translation that stays faithful to the published version.*

On the morning of November 14, I attended FCAJ x FPTU: Cloud Mastery 1. In the "The Core of Agent" session there was a slide about Amazon Bedrock AgentCore Memory that I felt deserved a post of its own, because it explains a situation that almost anyone building agents runs into. I redrew that diagram and numbered each data flow (Figure 1) so it is easier to follow.

**The scenario:** you are building a restaurant-recommendation chatbot that uses AgentCore Memory with the User Preference strategy enabled. A user writes: "I'm vegetarian, please don't suggest barbecue places." A few seconds later they ask another question, and the agent calls `RetrieveMemoryRecords` to look up their food preferences... and gets no results. The agent goes on to suggest a barbecue place anyway.

The first instinct is usually to suspect a misconfigured strategy or a weak extraction model. But the cause is more likely something simpler: the long-term memory has not been created yet. To understand why, let us walk through the five flows in the diagram.

### 1. Two memory tiers and five data flows

AgentCore Memory has two tiers:

- **Short-term memory:** stores each interaction verbatim as an event (messages, agent state), associated with an `actorId` (the user/agent) and a `sessionId` (the session).
- **Long-term memory:** stores memory records, that is, information distilled from events, such as preferences, important facts, and session summaries. Each type of record is produced by a strategy: Semantic (facts, knowledge), User preference (preferences), Summary (session summaries), Episodic (experiences organized by "episode"); you can also define your own custom strategies.

![AgentCore Memory data-flow diagram](/images/3-Blog/blog1_figure1.png)

*Figure 1. AgentCore Memory data-flow diagram (five numbered flows), redrawn from the "AgentCore Memory" slide at FCAJ x FPTU: Cloud Mastery 1 (November 14, 2026) and the AWS documentation*

Data moves through five flows:

- **(1) `CreateEvent`:** the agent writes messages and state to short-term memory. Synchronous: once written, the data is available.
- **(2) `ListEvents`:** the agent reads back the recent conversation from short-term memory. Synchronous.
- **(3) Automatic extraction:** a background module reads new events, extracts information (extraction), merges it with existing records (consolidation), and then writes the result to long-term memory. This flow is **ASYNCHRONOUS**.
- **(4) `RetrieveMemoryRecords`:** the agent performs a semantic search over long-term memory within a namespace. Synchronous, but it only sees what flow (3) has finished processing.
- **(5) Direct ingestion:** loads existing data into long-term memory without going through a conversation.

### 2. Why does the agent "forget"?

The key lies in flow (3). The AWS documentation states clearly that long-term memory extraction and consolidation is an asynchronous background process, that it can take a minute or more before information from a new conversation becomes retrievable, and that applications need to account for this delay.

In the scenario above, the "vegetarian" message reached short-term memory immediately through flow (1), but flow (3) had not finished yet, so flow (4) found no records. The agent only queried long-term memory, so it found nothing. This is not a bug; it is by design.

![The "forgetting agent" scenario in AgentCore Memory](/images/3-Blog/blog1_figure2.png)

*Figure 2. The "forgetting agent" scenario: short-term memory is written synchronously, while long-term memory is extracted asynchronously; the fix is to use ListEvents (short-term) within the session and RetrieveMemoryRecords (long-term) in later sessions*

One more pitfall that few people notice: only events created **AFTER** a strategy reaches the `ACTIVE` state are sent for extraction. If you add a strategy to a memory that is already in use, earlier conversations will not be processed automatically.

### 3. Design approach: read both tiers

Instead of relying only on flow (4), the agent should combine flows (2) and (4) in its context:

- **Within the same session:** fetch the last N turns with `ListEvents` (2). Whatever the user just said is always available here.
- **From previous sessions:** use `RetrieveMemoryRecords` (4) on the user's namespace to fetch the preferences and facts that have already been extracted.

Pseudocode for each turn:

```python
# (2) Immediate context
recent = ListEvents(memoryId, actorId, sessionId, maxResults=10)

# (4) Long-term knowledge that has already been extracted
facts = RetrieveMemoryRecords(memoryId, namespace=<strategy namespace>, searchQuery="food preferences", topK=5)

# Call the model with both sources
answer = Model(prompt + recent + facts + question)

# (1) Save the new turn; flow (3) will update long-term memory in the background
CreateEvent(memoryId, actorId, sessionId, [user_message, answer])
```

With this approach, short-term memory "holds on to" the preference the user just stated for the current session, while long-term memory catches up for use in later sessions.

### 4. Flow (5): when you do not want to wait for conversations

On the original slide, this flow is labeled "Batch API for long-term memory hydration". There are two options:

- **`BatchCreateMemoryRecords`:** directly creates memory records from content you prepare yourself (up to 100 records per request) and attaches them to the namespace you choose. This fits cases where you already have customer profiles or data from a legacy system and want the agent to "know" it from the start.
- **`IngestData`:** pushes content (conversations, or JSON such as behavior logs) straight into the extraction pipeline without storing it as short-term events. This still goes through the extraction step, so it is still asynchronous; according to the documentation, records typically appear within a few seconds to a few minutes, depending on the size of the content and the strategy configuration.

### 5. Checklist when the agent "doesn't remember"

- Have you waited for flow (3) to finish? Long-term memory always has a delay.
- Were the events created after the strategy became `ACTIVE`?
- Does the namespace used for retrieval in flow (4) match the namespace the strategy writes to?
- Is the `actorId` consistent across sessions for the same user?
- Did an extraction job fail? You can monitor the `FailedExtraction` metric in CloudWatch, find failed jobs with `ListMemoryExtractionJobs`, and rerun them with `StartMemoryExtractionJob`.

### Conclusion

AgentCore Memory clearly separates "remember now" from "remember for long": short-term memory is the synchronous path for immediate context, and long-term memory is the asynchronous path for durable knowledge across sessions. Once you understand that flow (3) is asynchronous, you will no longer be surprised when an agent "forgets" what the user just said, and you can design your agent correctly from the start.

*Note: AgentCore is evolving quickly, so check the latest documentation before applying this in production.*

### References

- [Memory types](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory-types.html)
- [Save and retrieve insights](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/long-term-saving-and-retrieving-insights.html)
- [Built-in strategies](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/built-in-strategies.html)
- [Ingest content into long-term memory](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/long-term-ingest-data.html)
- [BatchCreateMemoryRecords API](https://docs.aws.amazon.com/bedrock-agentcore/latest/APIReference/API_BatchCreateMemoryRecords.html)
- [AWS ML Blog – Amazon Bedrock AgentCore Memory](https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-agentcore-memory-building-context-aware-agents/)

#AWS #AmazonBedrock #BedrockAgentCore #AgentCoreMemory #AIAgent #AWSStudyGroup #FCAJ
