# 9. Semantic Caching

## Core Idea

Semantic Caching is an advanced LLM cost optimization technique that caches responses based on **semantic meaning** rather than exact text matching.

Unlike traditional response caching, which only works when two queries are identical, semantic caching uses **vector embeddings** to determine whether a new query is sufficiently similar to a previously answered one.

For example:

| User Query | Cached Query | Cache Hit? |
|------------|-------------|------------|
| What is OAuth? | Explain OAuth | ✅ Yes |
| Describe OAuth authentication | Explain OAuth | ✅ Yes |
| What is JWT? | Explain OAuth | ❌ No |

This significantly increases cache hit rates while reducing:

- API costs
- Response latency
- LLM inference load
- Rate-limit consumption

---

# Why Semantic Caching?

Many users ask the same question in different ways.

For example:

```
What is OAuth?
```

```
Explain OAuth.
```

```
Can you tell me about OAuth?
```

```
How does OAuth work?
```

Although these are different strings, they have nearly identical meaning.

A normal cache treats them as different requests.

Semantic caching recognizes their similarity and serves the same cached response.

---

# How It Works

The typical semantic cache pipeline is:

```
                User Query
                     │
                     ▼
          Generate Embedding
                     │
                     ▼
       Search Vector Cache (ANN)
                     │
                     ▼
      Similarity Above Threshold?
           /                   \
         Yes                   No
         │                     │
         ▼                     ▼
 Return Cached          Call LLM
 Response                   │
                            ▼
                   Store Response + Embedding
```

---

# Implementation Steps

1. Embed every incoming query using a fast embedding model.
2. Store:
   - Query embedding
   - Original query
   - LLM response
   - Metadata (timestamp, model, tenant, etc.)
3. Search the vector database for the nearest cached query.
4. Compute cosine similarity.
5. If similarity exceeds a predefined threshold, return the cached response.
6. Otherwise:
   - Call the LLM
   - Cache the new response
   - Store its embedding for future reuse

---

# Example

### Cached Query

```
Explain OAuth authentication
```

Embedding:

```
[0.14, 0.82, -0.33, ...]
```

Cached Response:

```
OAuth is an authorization framework...
```

---

### Incoming Query

```
What is OAuth?
```

Embedding:

```
[0.13, 0.80, -0.35, ...]
```

Cosine similarity:

```
0.96
```

Threshold:

```
0.92
```

Since:

```
0.96 > 0.92
```

The cached response is returned without invoking the LLM.

---

# Similarity Metrics

The most common similarity measures are:

| Metric | Use Case |
|---------|----------|
| Cosine Similarity | Most common for embeddings |
| Dot Product | Some embedding models |
| Euclidean Distance | Less common |
| Manhattan Distance | Rarely used |

Cosine similarity is the industry standard.

Formula:

```
similarity(A, B) =
(A · B) / (||A|| ||B||)
```

Typical values:

| Similarity | Interpretation |
|------------|---------------|
| 1.00 | Identical meaning |
| 0.95 | Extremely similar |
| 0.90 | Similar |
| 0.80 | Somewhat related |
| <0.75 | Usually different |

---

# Choosing a Similarity Threshold

Threshold selection determines cache quality.

| Threshold | Effect |
|-----------|--------|
| 0.99 | Almost exact meaning; very safe but fewer hits |
| 0.95 | Conservative |
| 0.92 | Good balance (common starting point) |
| 0.90 | Higher hit rate with slight risk |
| <0.85 | May return incorrect responses |

Thresholds should be tuned using real production traffic.

---

# Example Python Workflow

```python
embedding = embed(query)

nearest = vector_store.search(embedding)

if nearest.similarity > 0.92:
    return nearest.response

response = llm(query)

vector_store.insert(
    embedding,
    query,
    response
)

return response
```

---

# Where to Store Embeddings

Popular vector databases include:

- FAISS
- Qdrant
- Milvus
- Pinecone
- Weaviate
- ChromaDB
- Redis Vector Search
- pgvector (PostgreSQL)

---

# Benefits

## Higher Cache Hit Rate

Traditional response caches only work for identical queries.

Semantic caching also works for paraphrased queries.

---

## Lower Costs

Every cache hit avoids an LLM API call, directly reducing inference costs.

---

## Reduced Latency

Returning a cached response from a vector store is typically much faster than generating a new response.

---

## Better User Experience

Users receive faster responses, even when phrasing questions differently.

---

## Reduced Infrastructure Load

Fewer LLM requests mean lower GPU utilization and improved scalability.

---

# Challenges

## Threshold Tuning

If the similarity threshold is too low, unrelated queries may receive incorrect cached responses.

If it is too high, many semantically equivalent queries will miss the cache.

---

## Embedding Drift

Changing the embedding model can alter vector representations, potentially invalidating existing cache entries.

---

## Storage Growth

As the cache grows, embedding storage and search indexes require maintenance and periodic cleanup.

---

## Response Freshness

Cached responses may become outdated if the underlying knowledge changes, requiring cache invalidation or expiration policies.

---

# Best Practices

- Use a fast, lightweight embedding model.
- Normalize embeddings if required by the model.
- Start with a cosine similarity threshold around **0.92** and tune using production data.
- Apply TTLs (Time-To-Live) to expire stale cache entries.
- Include metadata (tenant, model version, language) to avoid incorrect cache reuse.
- Monitor cache hit rate and periodically evaluate response quality.
- Combine semantic caching with exact-match response caching for maximum efficiency.

---

# Comparison with Response Caching

| Feature | Response Caching | Semantic Caching |
|----------|------------------|------------------|
| Exact text match | ✅ Yes | ❌ No |
| Paraphrase support | ❌ No | ✅ Yes |
| Requires embeddings | ❌ No | ✅ Yes |
| Uses vector database | ❌ No | ✅ Yes |
| Higher cache hit rate | ❌ Lower | ✅ Higher |
| More computational overhead | ✅ Low | ⚠ Moderate |

---

# Trade-offs

| Advantage | Disadvantage |
|------------|--------------|
| Higher cache hit rates | Requires embedding generation |
| Lower LLM costs | Additional vector storage |
| Faster responses | Threshold tuning is non-trivial |
| Better scalability | Embedding computation adds slight latency |
| Supports paraphrased queries | Risk of incorrect cache hits if threshold is too low |

---

# When to Use Semantic Caching

Semantic caching is particularly effective for:

- Customer support chatbots
- Enterprise knowledge assistants
- Documentation Q&A systems
- FAQ bots
- Internal copilots
- Retrieval-Augmented Generation (RAG) systems
- Help desk automation
- API documentation assistants

It is especially valuable when users frequently ask the same questions using different wording.

---

# Research Papers

1. **GPTCache: Semantic Cache for LLM Applications (2023)**  
   https://github.com/zilliztech/GPTCache

2. **Approximate Nearest Neighbor Search: A Survey**  
   https://arxiv.org/abs/2306.07426

3. **FAISS: A Library for Efficient Similarity Search and Clustering of Dense Vectors**  
   https://arxiv.org/abs/1702.08734

4. **BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding** (Introduced contextual embeddings that underpin semantic similarity)
   https://arxiv.org/abs/1810.04805

5. **Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks**
   https://arxiv.org/abs/1908.10084

---

# Articles and Blogs

## OpenAI

- Embeddings Guide  
  https://platform.openai.com/docs/guides/embeddings

- Embeddings API Reference  
  https://platform.openai.com/docs/api-reference/embeddings

---

## Anthropic

- Prompt Caching Documentation  
  https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching

---

## Pinecone

- Semantic Search Explained  
  https://www.pinecone.io/learn/semantic-search/

- What is a Vector Database?  
  https://www.pinecone.io/learn/vector-database/

---

## Qdrant

- Semantic Search Documentation  
  https://qdrant.tech/documentation/

- Vector Similarity Search  
  https://qdrant.tech/documentation/concepts/search/

---

## Weaviate

- Semantic Search Concepts  
  https://weaviate.io/developers/weaviate/concepts/search

---

## Redis

- Semantic Caching with Redis LangCache  
  https://redis.io/docs/latest/develop/ai/langcache/

---

## GPTCache

- Official Documentation  
  https://gptcache.readthedocs.io/

- GitHub Repository  
  https://github.com/zilliztech/GPTCache

---

## LangChain

- Caching Documentation  
  https://python.langchain.com/docs/integrations/llm_caching/

---

## Microsoft

- Vector Search Overview (Azure AI Search)  
  https://learn.microsoft.com/azure/search/vector-search-overview

---

# Summary

Semantic caching extends traditional response caching by using **vector embeddings** to identify semantically similar queries rather than relying on exact text matches. By retrieving cached responses for paraphrased or equivalent questions, it significantly reduces LLM inference costs, lowers latency, and improves scalability. Although it introduces additional complexity through embedding generation, vector storage, and similarity threshold tuning, it is one of the most effective optimization techniques for production-grade LLM applications where repeated questions are common.