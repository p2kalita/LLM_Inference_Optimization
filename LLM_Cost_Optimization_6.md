# RAG (Retrieval-Augmented Generation)

## Overview

**Retrieval-Augmented Generation (RAG)** is one of the most effective techniques for reducing LLM inference costs while simultaneously improving factual accuracy.

Instead of sending an entire knowledge base or lengthy document to the LLM, RAG first retrieves only the most relevant information required to answer the user's query. The retrieved content is then included in the prompt, significantly reducing input tokens while providing the model with grounded, context-specific knowledge.

This approach is widely used in enterprise AI systems, customer support chatbots, document assistants, legal AI, healthcare assistants, and internal knowledge search.

---

# Why RAG Reduces Cost

Without RAG, applications often send hundreds of pages or entire documents to the model.

## Without RAG

```text
User Question:
"What is the refund policy for enterprise customers?"

Prompt Sent:
Entire Employee Handbook
Entire Terms & Conditions
Entire FAQ
Entire Policy Manual

≈ 40,000 tokens
```

## With RAG

```text
User Question
      │
      ▼
Vector Search
      │
Retrieve Top-3 Relevant Chunks
      │
      ▼
LLM Prompt

≈ 900 tokens
```

The model processes only the information needed to answer the query.

---

# How RAG Works

```text
                Documents
                    │
                    ▼
            Document Chunking
                    │
                    ▼
              Text Embeddings
                    │
                    ▼
              Vector Database
                    │
────────────────────┼────────────────────
                    │
              User Question
                    │
                    ▼
           Query Embedding Model
                    │
                    ▼
          Similarity Search (Top-K)
                    │
                    ▼
         Retrieved Relevant Chunks
                    │
                    ▼
            Prompt Construction
                    │
                    ▼
                Large Language Model
                    │
                    ▼
                 Final Answer
```
---
# Step 1 — Document Chunking
Large documents are split into manageable pieces before embedding.

Example:

Original document

```

Page 1
Company Leave Policy

Page 2
Annual Leave Rules

Page 3
Sick Leave

Page 4
Maternity Leave

...
```
After chunking

```

Chunk 1
Introduction

Chunk 2
Annual Leave

Chunk 3
Sick Leave

Chunk 4
Maternity Leave

Chunk 5
Remote Work

...
```

Common chunk sizes:

| Chunk Size     | Use Case                  |
| -------------- | ------------------------- |
| 200 tokens     | FAQ                       |
| 300–500 tokens | General RAG               |
| 500–800 tokens | Technical Docs            |
| 1000+ tokens   | Long contextual reasoning |

Smaller chunks improve retrieval precision but may lose context, while larger chunks preserve context but increase prompt size and cost.

---
# Step 2 — Generate Embeddings
Each chunk is converted into a high-dimensional vector representation.

Example:

```

Chunk

"Our refund policy allows..."

↓

Embedding

[0.12, -0.93, 0.56, ...]
```


Popular embedding models include:

- OpenAI text-embedding-3-small
- OpenAI text-embedding-3-large
- BAAI BGE Series
- Sentence Transformers (all-MiniLM-L6-v2)
- NVIDIA NV-Embed
- E5 Embeddings
- Jina Embeddings
- Cohere Embed
- Voyage AI Embeddings

Embedding is performed only once during indexing, making query-time retrieval efficient.


---
# Step 3 — Store Embeddings

Embeddings are stored in a vector database for efficient similarity search.

Popular vector databases:

| Vector Database | Open Source | Cloud      | Notes                      |
| --------------- | ----------- | ---------- | -------------------------- |
| FAISS           | ✅           | ❌          | Fast local search          |
| ChromaDB        | ✅           | ❌          | Simple development         |
| Qdrant          | ✅           | ✅          | High-performance filtering |
| Milvus          | ✅           | ✅          | Large-scale deployments    |
| Weaviate        | ✅           | ✅          | Graph + vector search      |
| pgvector        | ✅           | PostgreSQL | SQL integration            |
| Pinecone        | ❌           | ✅          | Fully managed              |
| Vespa           | ✅           | ✅          | Web-scale search           |
| LanceDB         | ✅           | ❌          | Embedded vector database   |

---
# Step 4 — User Query

Suppose the user asks:

```
"What is the maternity leave policy?"
```

The question is embedded using the same embedding model.

```
Question

↓

Embedding Vector

↓

Vector Search
```
---

# Step 5 — Similarity Search

The vector database finds the closest document chunks.

Instead of searching every word literally, semantic similarity is used.

Example
```
User Query

"What is maternity leave?"

Retrieved:

✓ Chunk 42
"Maternity Leave"

✓ Chunk 71
"Parental Leave"

✓ Chunk 15
"Benefits Overview"
```
Only these chunks are returned.

---
# Step 6 — Prompt Construction

Instead of:
```
Entire Company Handbook

50,000 tokens
```
The prompt becomes:
```
System Prompt

Retrieved Context

Chunk 42

Chunk 71

Chunk 15

User Question
```
Often:
```
500–1500 tokens
```

instead of
```
30k–100k tokens
```
---
# Step 7 — LLM Generates the Answer
The model answers using only the retrieved context.

Example:
```
Question:
What is maternity leave?

Retrieved Context:
Employees receive 26 weeks paid maternity leave...

Answer:
Employees are eligible for 26 weeks of paid maternity leave...
```

This grounding reduces hallucinations and improves factual consistency.
---
# Example Cost Comparison
Assume GPT-4o input pricing.

Without RAG:
```
Knowledge Base

80,000 tokens

Question

100 tokens

Total

≈80,100 tokens
```

With RAG:
```
Top-3 Chunks

900 tokens

Question

100 tokens

Total

≈1000 tokens
```

Cost Reduction
```

80,100
↓

1,000

≈98.75% fewer input tokens
```
This translates directly into lower inference costs and faster responses.

---
# Choosing Top-K

Top-K determines how many chunks are retrieved.

| Top-K | Benefits       | Drawbacks                                        |
| ----- | -------------- | ------------------------------------------------ |
| 1     | Cheapest       | May miss context                                 |
| 3     | Good balance   | Most common choice                               |
| 5     | Better recall  | Higher cost                                      |
| 10    | Comprehensive  | Large prompts                                    |
| 20+   | Maximum recall | Expensive and may include irrelevant information |

Most production systems use Top-K = 3–5.
---
# Chunk Size Trade-offs

| Chunk Size | Precision | Context  | Cost   |
| ---------- | --------- | -------- | ------ |
| Small      | High      | Low      | Low    |
| Medium     | Balanced  | Balanced | Medium |
| Large      | Lower     | High     | High   |

A chunk size of 300–500 tokens is often a good starting point for general-purpose RAG systems.
---
# Common Retrieval Techniques

### 1. Dense Retrieval

Uses vector similarity search based on embeddings. Best for semantic understanding.

### 2. Sparse Retrieval

Uses keyword-based methods such as BM25. Effective for exact keyword matches.

### 3. Hybrid Search

Combines dense and sparse retrieval to improve both recall and precision.

### 4. Metadata Filtering

Filters documents by metadata (e.g., date, department, document type) before similarity search.

### 5. Parent-Child Retrieval

Retrieves small child chunks but expands them into larger parent sections for richer context.

### 6. Multi-Query Retrieval

Generates multiple reformulations of the user's query to improve retrieval coverage.

### 7. Contextual Compression

Retrieves larger chunks and then compresses them using another model before sending them to the LLM.

### 8. Reranking

After initial retrieval, a reranker (e.g., cross-encoder or LLM-based reranker) reorders results so the most relevant chunks are passed to the LLM.
---

# Implementation Steps

1. Chunk your documents into semantic sections.
2. Generate embeddings using an embedding model.
3. Store embeddings in a vector database.
4. Embed the user's query.
5. Retrieve the Top-K most relevant chunks.
6. Optionally rerank retrieved results.
7. Construct the prompt using only retrieved context.
8. Generate the final answer.

---

# Popular Embedding Models

- OpenAI text-embedding-3-small
- OpenAI text-embedding-3-large
- BAAI BGE Series
- Sentence Transformers (all-MiniLM-L6-v2)
- E5 Embeddings
- NVIDIA NV-Embed
- Cohere Embed
- Voyage AI Embeddings
- Jina Embeddings

---

# Popular Vector Databases

| Database | Open Source | Cloud |
|-----------|-------------|-------|
| FAISS | ✅ | ❌ |
| ChromaDB | ✅ | ❌ |
| Qdrant | ✅ | ✅ |
| Milvus | ✅ | ✅ |
| Weaviate | ✅ | ✅ |
| pgvector | ✅ | PostgreSQL |
| Pinecone | ❌ | ✅ |
| LanceDB | ✅ | ❌ |
| Vespa | ✅ | ✅ |

---

# Choosing Chunk Size

| Chunk Size | Best For |
|------------|----------|
| 200 tokens | FAQs |
| 300–500 tokens | General RAG |
| 500–800 tokens | Technical Documentation |
| 1000+ tokens | Long-form reasoning |

---

# Choosing Top-K

| Top-K | Notes |
|-------|------|
| 1 | Lowest cost |
| 3 | Recommended default |
| 5 | Better recall |
| 10 | Higher context and cost |

---

# Benefits

- Significant reduction in prompt tokens
- Lower inference cost
- Better factual accuracy
- Reduced hallucinations
- Dynamic knowledge updates
- Citation support
- Scalable to millions of documents

---

# Trade-offs

| Advantage                                             | Drawback                                               |
| ----------------------------------------------------- | ------------------------------------------------------ |
| Significant reduction in LLM input tokens             | Retrieval quality becomes a critical dependency        |
| Lower inference cost                                  | Requires a vector database and embedding pipeline      |
| More accurate and grounded responses                  | Poor chunking can reduce answer quality                |
| Scales to large document collections                  | Additional retrieval latency                           |
| Supports dynamic knowledge updates without retraining | Requires monitoring and tuning of retrieval parameters |


---

# Best Practices

- Use semantic chunking whenever possible.
- Start with 300–500 token chunks.
- Use Top-K between 3 and 5.
- Apply reranking for better retrieval quality.
- Include metadata for citations.
- Measure Recall@K and MRR.
- Cache embeddings and frequently retrieved chunks.

---

# Research Papers

## 1. Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks
https://arxiv.org/abs/2005.11401

## 2. REALM: Retrieval-Augmented Language Model Pre-Training
https://arxiv.org/abs/2002.08909

Microsoft Research:
https://www.microsoft.com/en-us/research/publication/realm-retrieval-augmented-language-model-pre-training/

## 3. Leveraging Passage Retrieval with Generative Models for Open Domain Question Answering (FiD)
https://arxiv.org/abs/2007.01282

## 4. Atlas: Few-shot Learning with Retrieval Augmented Language Models
https://arxiv.org/abs/2208.03299

## 5. Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection
https://arxiv.org/abs/2310.11511

## 6. Retrieval-Augmented Generation for Large Language Models: A Survey
https://arxiv.org/abs/2312.10997

---

# Articles & Blogs

## OpenAI Cookbook
https://cookbook.openai.com/

## LangChain RAG Tutorials
https://python.langchain.com/docs/tutorials/rag/

## LangChain Retrieval
https://python.langchain.com/docs/concepts/retrieval/

## LlamaIndex Documentation
https://docs.llamaindex.ai/

## Pinecone Learn
https://www.pinecone.io/learn/

## Pinecone RAG Guide
https://www.pinecone.io/learn/retrieval-augmented-generation/

## Qdrant Documentation
https://qdrant.tech/documentation/

## Weaviate Documentation
https://weaviate.io/developers/weaviate

## Milvus Documentation
https://milvus.io/docs

## ChromaDB Documentation
https://docs.trychroma.com/

## pgvector
https://github.com/pgvector/pgvector

## Hugging Face Learn
https://huggingface.co/learn

## Transformers RAG Documentation
https://huggingface.co/docs/transformers/model_doc/rag

## Microsoft Azure AI
https://learn.microsoft.com/azure/ai-foundry/

## NVIDIA Developer Blog
https://developer.nvidia.com/blog/

## Anthropic Engineering
https://www.anthropic.com/engineering

## Cohere Documentation
https://docs.cohere.com/

---

# Additional Advanced RAG Papers

- CRAG — https://arxiv.org/abs/2401.15884
- GraphRAG — https://www.microsoft.com/en-us/research/project/graphrag/
- RAPTOR — https://arxiv.org/abs/2401.18059
- RAFT — https://arxiv.org/abs/2403.10131
- LongRAG — https://arxiv.org/abs/2406.15319
- LightRAG — https://arxiv.org/abs/2410.05779

---

# Conclusion

RAG has become the de facto architecture for building enterprise-grade LLM applications because it dramatically reduces token usage while improving factual accuracy. By retrieving only the most relevant information at query time, organizations can lower inference costs, reduce hallucinations, and keep AI systems up to date without retraining large language models.
