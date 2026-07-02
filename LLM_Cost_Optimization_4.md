# 4: Conversation Summarization

## Overview

As conversations with Large Language Models (LLMs) grow longer,
repeatedly sending the entire conversation history becomes increasingly
expensive. Every additional message consumes input tokens, leading to
higher inference costs, increased latency, and reduced efficiency.

**Conversation Summarization** addresses this problem by periodically
compressing older conversation history into a concise summary. Instead
of carrying the full transcript throughout the session, the application
replaces historical messages with a compact memory representation while
retaining the most recent interactions.

This technique significantly reduces token usage while preserving the
essential context needed for coherent conversations.

------------------------------------------------------------------------

## Why Conversation Summarization Matters

Long-running applications such as:

-   AI Chatbots
-   Customer Support Assistants
-   Coding Agents
-   AI Tutors
-   Personal Assistants

may accumulate hundreds or even thousands of conversation turns.

Without summarization:

``` text
System Prompt
+
Conversation Turn 1
Conversation Turn 2
...
Conversation Turn 250
+
Current User Query
```

Every request resends the entire history.

With summarization:

``` text
System Prompt
+
Conversation Summary
+
Recent Conversation
+
Current User Query
```

------------------------------------------------------------------------

## How Conversation Summarization Works

1.  Track conversation length using token count or message count.
2.  When a threshold is reached, trigger summarization.
3.  Use a lightweight LLM to summarize older conversation history.
4.  Store the generated summary as persistent memory.
5.  Remove summarized messages from the active context.
6.  Keep only the summary and the most recent conversation turns.

------------------------------------------------------------------------

## Common Summarization Strategies

### 1. Turn-Based Summarization

Summarize after every fixed number of conversation turns (e.g., every
20, 50, or 100 messages).

**Advantages**

-   Simple
-   Predictable

**Limitations**

-   May summarize unnecessarily.

------------------------------------------------------------------------

### 2. Token-Based Summarization

Trigger summarization when prompt size exceeds a token threshold.

``` text
Context Window = 128K tokens
Current Prompt = 100K tokens
Threshold = 90K tokens
→ Trigger summarization
```

------------------------------------------------------------------------

### 3. Hybrid Memory

Store:

-   Conversation Summary
-   Last 5--10 raw conversation turns

Example:

``` text
Conversation Summary

• User is building an invoice-processing chatbot.
• Uses Qdrant.
• Uses LiteLLM.
• Semantic caching implemented.

Recent Messages

User: How should I optimize retrieval?
Assistant: Use hybrid search...

Current Question:
How can I reduce inference latency?
```

------------------------------------------------------------------------

### 4. Hierarchical Summarization

``` text
Messages
 ↓
Summary A
 ↓
Summary B
 ↓
Summary C
 ↓
Master Summary
```

Useful for extremely long-running AI agents.

------------------------------------------------------------------------

## Implementation Steps

1.  Define a turn-count or token-count threshold.
2.  Use a lightweight model (GPT-4o Mini, Gemini Flash, Llama 3 8B,
    etc.).
3.  Summarize:
    -   User goals
    -   Decisions
    -   Constraints
    -   Key facts
    -   Outstanding tasks
4.  Replace old messages with the summary.
5.  Retain recent conversation turns.
6.  Refresh summaries periodically.

------------------------------------------------------------------------

## Benefits

-   Lower inference costs
-   Lower token usage
-   Faster responses
-   Longer conversations
-   Better scalability
-   Improved context utilization

------------------------------------------------------------------------

## Trade-offs

-   Lossy compression
-   Nuances may be omitted
-   Exact wording is lost
-   Summarization itself incurs some cost

------------------------------------------------------------------------

## Best Practices

-   Trigger using token count rather than turn count.
-   Use inexpensive models for summarization.
-   Preserve recent raw messages.
-   Store structured summaries.
-   Periodically regenerate summaries.
-   Combine with retrieval-based memory and context trimming.

------------------------------------------------------------------------

## Example

### Without Summarization

``` text
System Prompt
Conversation (120 turns)
Current User Query
```

Input Tokens: **18,500**

### With Summarization

``` text
System Prompt
Conversation Summary (400 tokens)
Last 6 messages
Current User Query
```

Input Tokens: **2,300**

This reduces prompt size by nearly **88%**.

------------------------------------------------------------------------

## Comparison

  Technique                    Purpose                      Keeps Raw History   Information Loss
  ---------------------------- ---------------------------- ------------------- ------------------
  Context Trimming             Remove irrelevant messages   Partial             Low
  Conversation Summarization   Compress older history       No                  Moderate
  Hybrid Memory                Summary + Recent Turns       Yes                 Low--Moderate

------------------------------------------------------------------------

# Recommended Reading

## Engineering Blogs

### Anthropic -- Building Effective AI Agents

https://www.anthropic.com/engineering/building-effective-agents

### LangChain -- Conversation Summary Memory

https://python.langchain.com/docs/modules/memory/types/summary/

### Microsoft Semantic Kernel -- Chat History Reducers

https://learn.microsoft.com/semantic-kernel/

------------------------------------------------------------------------

## Research Papers

### ReSum: Unlocking Long-Horizon Search Intelligence via Context Summarization

https://arxiv.org/abs/2509.13313

### LongMem: Augmenting Language Models with Long-Term Memory

https://arxiv.org/abs/2306.07174

------------------------------------------------------------------------

## Open Source

### Mem0

https://mem0.ai/

Production-ready memory layer for AI applications that combines
summarization, retrieval, and persistent memory.

------------------------------------------------------------------------

## Key Takeaway

Conversation Summarization is a core optimization technique for
production LLM systems. By replacing lengthy conversation histories with
concise summaries while preserving recent interactions, organizations
can significantly reduce inference costs, extend conversation length,
and maintain coherent user experiences. Combined with context trimming,
retrieval-based memory, and semantic caching, it enables scalable and
cost-efficient LLM applications.
