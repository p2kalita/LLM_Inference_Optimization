
# 7. Response Caching

### Core Idea

Response Caching is a cost optimization technique where the system stores previously generated LLM responses and reuses them for identical or highly similar queries instead of invoking the model again.

Since LLM inference is often the most expensive part of an application, serving a cached response can reduce:

- API costs
- Response latency
- Infrastructure load
- Rate-limit usage

This is one of the simplest yet most effective optimizations for production LLM systems. 



# Why Response Caching?

Many users ask the same or very similar questions.

For example:
```
"What is RAG?"
```
A few minutes later another user asks:
```
what is rag
```
Or
```
What is Retrieval Augmented Generation?
```
Without caching:
```
User
   │
   ▼
LLM API
   │
   ▼
Response
```
Every request costs money.

With caching:
```
User
   │
   ▼
Cache Lookup
   │
 ┌─┴────────────┐
 │              │
Hit            Miss
 │              │
 ▼              ▼
Cached      Call LLM
Response       │
               ▼
         Store in Cache
```
Only cache misses invoke the LLM.

# How It Works

A typical response caching workflow consists of the following steps.

### Step 1 — Normalize the Query

Users may write the same question in different formats.

Example:
```
"What is RAG?"
```
```
"what is rag"
```
```
"   What is RAG   "
```

Normalize by:

- Lowercasing
- Removing extra whitespace
- Removing punctuation (optional)
- Unicode normalization
- Standardizing numbers/dates if needed

Example:
```
Input:
"What is RAG?"

↓

Normalized:

what is rag
```
---
### Step 2 — Generate a Cache Key

Create a unique identifier.

Common approaches:
```
SHA-256(query)
```
or
```
MD5(query)
```
Example:
```
what is rag

↓

a8d61f82e...
```
This becomes the cache key.

---
### Step 3 — Lookup Cache

Before calling the LLM:
```
cache.get(hash)
```
Possible outcomes:


### Cache Hit
```
Cache

hash

↓

response found
```
Return immediately.

No API call.

No token cost.

---
### Cache Miss
```
Cache

↓

Nothing found
```
Call the LLM.

---
### Step 4 — Call the LLM
```
LLM

↓

Generate answer
```
---

### Step 5 — Store Result
```
cache.set(
    hash,
    response,
    ttl=24h
)
```

Future requests reuse this answer.




# Cache Architecture

```
              User Query
                   │
                   ▼
          Normalize Query
                   │
                   ▼
            Generate Hash
                   │
                   ▼
             Cache Lookup
           ┌───────────────┐
           │               │
       Cache Hit      Cache Miss
           │               │
           ▼               ▼
 Return Cached      Call LLM API
    Response             │
                         ▼
                 Generate Response
                         │
                         ▼
                  Store in Cache
                         │
                         ▼
                    Return Result
```
---

# Cache Layers

Several storage systems are commonly used.

| Cache            | Characteristics                           | Best Use Cases                     |
| ---------------- | ----------------------------------------- | ---------------------------------- |
| Redis            | Extremely fast, supports TTL, distributed | Most LLM production systems        |
| Memcached        | Lightweight in-memory cache               | High-speed ephemeral caching       |
| Local Memory     | Process-level cache                       | Small applications and development |
| SQLite           | Persistent local cache                    | Desktop or offline applications    |
| Disk-based Cache | Persistent but slower                     | Batch processing                   |
| CDN Edge Cache   | Global edge caching                       | Public chatbots and APIs           |

---

# Cache Key Design

A poor cache key leads to low hit rates.

Instead of:
```
Original Query
```
Use:
```
Normalized Query
```
Sometimes include additional context:
```
model_name

+

system_prompt_version

+

normalized_query
```
Example:
```
gpt-4.1

v3

what is rag
```
This prevents returning responses generated under different prompts or models.



# Choosing an Appropriate TTL

TTL (Time-To-Live) determines how long a cached response remains valid before it expires.

| Content Type          | Suggested TTL      |
| --------------------- | ------------------ |
| General knowledge     | Days to weeks      |
| Company documentation | Hours to days      |
| Product catalog       | Minutes to hours   |
| Stock prices          | Seconds to minutes |
| Weather               | Minutes            |
| News                  | Minutes to hours   |

Choosing TTL involves balancing freshness and cost savings.




# Cache Invalidation Strategies

Cache invalidation is one of the hardest parts of caching. Common strategies include:

### Time-Based Expiration (TTL)

Automatically remove entries after a fixed duration.
```
TTL = 24 hours
```
---
### Event-Based Invalidation

Clear relevant cache entries when underlying data changes.

Example:
```
Knowledge Base Updated

↓

Delete Related Cache Keys
```
---
### Version-Based Cache

Include a version number in the cache key.
```
KB Version = 5

↓

cache key

v5_query_hash
```
When the knowledge base updates to version 6, new cache keys are automatically generated, avoiding stale responses without manually deleting old entries.

---
### Manual Purge

Administrators explicitly remove cached entries when necessary.


# Example (Python + Redis)
```python
import hashlib
import redis

redis_client = redis.Redis(host="localhost", port=6379, decode_responses=True)


def normalize(query: str) -> str:
    """Normalize the user query before caching."""
    return " ".join(query.lower().strip().split())


def cache_key(query: str) -> str:
    """Generate a SHA-256 cache key for a normalized query."""
    normalized = normalize(query)
    return hashlib.sha256(normalized.encode()).hexdigest()


def get_cached_response(query: str):
    """Retrieve a cached response if it exists."""
    return redis_client.get(cache_key(query))


def store_response(query: str, response: str, ttl: int = 86400):
    """Store a response in Redis with a TTL (default: 24 hours)."""
    redis_client.setex(cache_key(query), ttl, response)
```


# Advanced Response Caching
### 1. Semantic Caching

Instead of requiring an exact query match, semantic caching retrieves cached responses for similar questions using embeddings and vector search.

Example:
```
What is RAG?

↓

Retrieval Augmented Generation?

↓

Explain RAG.
```
Although the wording differs, the intent is the same. A semantic cache can identify this similarity and reuse an existing response.

Benefits:

- Higher cache hit rate
- Better handling of paraphrased queries

Challenges:

- Requires embedding models and a vector database
- Similarity thresholds must be tuned to avoid incorrect matches
---

### 2. Partial Response Caching

Rather than caching an entire answer, cache reusable components.

Example:
```
Prompt

↓

Retrieve Context

↓

Generate Summary
```
The retrieved context or intermediate computations can be cached independently, reducing work for repeated requests.

---

### 3. RAG-Aware Caching

For Retrieval-Augmented Generation (RAG), include the retrieval context or document version in the cache key.
```
Cache Key

=

Query

+

Retrieved Document IDs

+

Knowledge Base Version
```
This prevents serving responses based on outdated documents after the knowledge base changes.

---

### 4. Multi-Level Caching

Use multiple cache layers to optimize both latency and scalability.
```
Application Memory Cache
        │
        ▼
Redis Cache
        │
        ▼
LLM API
```
Frequently accessed responses are served from the fastest available cache layer.



# Benefits
- Significant reduction in LLM API costs
- Faster response times (often milliseconds)
- Lower infrastructure and networking overhead
- Reduced API rate-limit usage
- Improved scalability for high-traffic applications
- Better user experience through lower latency



# Limitations
- Limited benefit if most queries are unique
- Risk of serving stale responses if invalidation is inadequate
- Lower cache hit rates for highly personalized queries
- Additional infrastructure (e.g., Redis) may be required
- Semantic caching introduces complexity and potential false matches
- Cache management and monitoring add operational overhead



# Best Practices

- Normalize queries before hashing to maximize cache hits.
- Include the model name, prompt version, and knowledge base version in cache keys.
- Use Redis or Memcached for low-latency distributed caching.
- Configure TTLs based on how frequently the underlying information changes.
- Implement event-driven or version-based invalidation for dynamic data sources.
- Monitor cache hit rate, miss rate, latency, and memory usage to optimize performance.
- Consider semantic caching for FAQ-style applications with many paraphrased queries.
- Avoid caching sensitive or user-specific responses unless they are isolated per user and encrypted where appropriate.
# Trade-offs
| Advantages                 | Disadvantages                              |
| -------------------------- | ------------------------------------------ |
| Reduces LLM API costs      | Benefits only repeated or similar queries  |
| Improves response latency  | Stale responses if invalidation is poor    |
| Reduces backend load       | Additional cache infrastructure required   |
| Increases throughput       | Cache consistency becomes more complex     |
| Lowers rate-limit pressure | Personalized responses are harder to cache |

# Research Papers
1. GPTCache: An Open-Source Semantic Cache for LLM Applications (2023)
https://github.com/zilliztech/GPTCache
2. Prompt Cache: Modular Attention Reuse for Low-Latency LLM Serving (2024)
https://arxiv.org/abs/2311.04934
3. CacheGen: KV Cache Compression and Streaming for Fast LLM Inference (2024)
https://arxiv.org/abs/2310.07240
4. CacheBlend: Fast Large Language Model Serving for RAG with Cached Knowledge Fusion (2024)
https://arxiv.org/abs/2405.16444
# Useful Blogs & Articles
- Redis – Caching Patterns
https://redis.io/docs/latest/develop/use/patterns/
- Redis – Caching Overview
https://redis.io/glossary/cache/
- GPTCache Documentation
https://gptcache.readthedocs.io/
- GPTCache GitHub Repository
https://github.com/zilliztech/GPTCache
- LangChain Caching Documentation
https://python.langchain.com/docs/how_to/llm_caching/
- OpenAI Prompt Caching Guide
https://platform.openai.com/docs/guides/prompt-caching
- Anthropic Prompt Caching Documentation
https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
- RedisVL Semantic Cache Documentation
https://redis.io/docs/latest/develop/ai/redisvl/user_guide/llmcache/
- Azure Cache for Redis Documentation
https://learn.microsoft.com/azure/azure-cache-for-redis/
- AWS ElastiCache Documentation
https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.html