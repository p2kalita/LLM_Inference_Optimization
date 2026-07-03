# 14. Query Classification

### Core Idea

**Query Classification** is an inference optimization technique where every incoming user request is first classified into a predefined category before any expensive reasoning occurs.

Instead of sending every request to the same LLM workflow, the system determines the nature of the query and routes it to the **cheapest pipeline capable of solving it**.

For example:

- FAQ → Static knowledge lookup
- Documentation Search → RAG pipeline
- Coding Question → Coding LLM
- Data Analysis → Tool + LLM
- Multi-step Task → Agent
- Simple Calculation → Deterministic function

The goal is to ensure that **not every request triggers expensive retrieval, planning, or agent execution.**

---

# Why Query Classification?

Many production AI applications waste significant compute because every request follows the same expensive pipeline.

Example:

```
User: What are your support hours?

Without Classification:

   User
    ↓
Large LLM
    ↓
  Agent
    ↓
Retriever
    ↓
Tool Calls
    ↓
  Answer
```

This may consume:

- 3-5 tool calls
- Retrieval
- Planning
- Thousands of tokens

Even though the answer is simply:

> "Support is available Monday–Friday, 9 AM–6 PM."

A classifier avoids this waste.

---

# How It Works

Instead of directly invoking the main AI system:

```
      Incoming Query
            │
            ▼
      Query Classifier
            │
    ┌───────┼───────────┐
    │       │           │
    ▼       ▼           ▼
   FAQ     RAG        Agent
    │       │           │
    ▼       ▼           ▼
Lookup   Retriever  Planning
```

The classifier identifies what type of problem the user has and routes it to the appropriate pipeline.

---

# Typical Pipeline Categories

| Query Type | Pipeline | Cost |
|------------|----------|------|
| FAQ | Static database | Very Low |
| Greeting | Template | Almost Free |
| Math | Calculator Tool | Very Low |
| Weather | Weather API | Very Low |
| Product Search | Search API | Low |
| Document Question | RAG | Medium |
| SQL Generation | Coding Model | Medium |
| Code Review | Coding LLM | Medium |
| Image Request | Vision/Image Model | Medium |
| Multi-step Task | Agent | High |
| Research | Agent + RAG | Very High |

---

# Example Workflow

Suppose users ask:

```
Q1:
What is your refund policy?

→ FAQ
→ Cached Answer
```

---

```
Q2:
Summarize this PDF.

→ Document Pipeline
→ OCR
→ Chunking
→ RAG
→ LLM
```

---

```
Q3:
Write Python code for BFS.

→ Coding Pipeline
→ Coding LLM
```

---

```
Q4:
Analyze my financial report and compare with industry benchmarks.

→ Agent
→ Tool Calls
→ Retrieval
→ Reasoning
```

Notice that only the final request actually requires an expensive agent.

---

# Classification Methods

Several approaches can be used depending on the complexity of your application.

## 1. Rule-Based Classification

For obvious and repetitive queries, simple rules are often sufficient.

Example:

```python
if "refund" in query.lower():
    return "faq"

if "weather" in query.lower():
    return "weather"

if "price" in query.lower():
    return "pricing"
```

### Advantages

- Extremely fast
- No LLM cost
- Easy to debug

### Limitations

- Difficult to scale
- Brittle for varied phrasing
- Requires manual maintenance

---

## 2. Keyword Matching

A slightly more flexible approach maps keywords to categories.

Example:

```
Keywords:

     invoice
     receipt
     payment
       tax

        ↓

 Finance Pipeline
```

This is suitable for many enterprise applications with predictable terminology.

---

## 3. Embedding-Based Classification

The query is converted into an embedding and compared against embeddings representing known categories.

```
Query Embedding

↓

Similarity Search

↓

Closest Category

↓

Pipeline
```

Useful when users express the same intent using different wording.

---

## 4. Small LLM Classification

A lightweight model determines the category before routing.

Example prompt:

```
Classify this query into exactly one category:

- FAQ
- Coding
- RAG
- Agent
- Math

Query:

"Generate SQL to join two tables."

Return only the category.
```

Only a few tokens are required, making this much cheaper than running the full pipeline.

---

## 5. Fine-Tuned Intent Classifier

Large-scale systems often train dedicated intent classification models.

Examples:

- DistilBERT
- MiniLM
- BERT
- FastText
- Sentence Transformers

Advantages:

- Very fast
- Low latency
- No LLM inference cost
- High throughput

---

# Example Architecture

```
                   User Query
                        │
                        ▼
             Query Classification
                        │
      ┌─────────────────┼─────────────────┐
      │                 │                 │
      ▼                 ▼                 ▼
 FAQ Pipeline      RAG Pipeline      Agent Pipeline
      │                 │                 │
      ▼                 ▼                 ▼
 Static DB      Vector Search          Planning
                                        Tools
                                        Memory
                                         LLM
```

Each pipeline is optimized independently.

---

# Implementation Steps

## Step 1: Identify Query Categories

Analyze your application logs to understand the common types of user requests.

Example categories:

- FAQs
- Customer support
- Code generation
- Document Q&A
- Search
- Summarization
- Translation
- Analytics
- Multi-step workflows

---

## Step 2: Build a Lightweight Classifier

Start with the simplest effective approach:

- Regex
- Keyword matching
- Embeddings
- Small LLM
- Fine-tuned classifier

Avoid using a large reasoning model for classification.

---

## Step 3: Route Queries

Map each category to the most appropriate pipeline.

Example:

```
       FAQ
        ↓

    Redis Cache

---------------------

Document Question
        ↓

    Retriever
        ↓

       LLM

---------------------

  Complex Workflow
        ↓

      Agent

        ↓

      Tools
```

---

## Step 4: Log Every Decision

Capture routing metadata such as:

- Query
- Predicted category
- Pipeline selected
- Response latency
- Token usage
- User feedback (optional)

Example:

```
Query:
"Generate Docker Compose"

Category:
Coding

Pipeline:
Coding LLM

Latency:
0.9 sec

Tokens:
640
```

These logs help identify incorrect routing decisions.

---

## Step 5: Refine Categories Over Time

User behavior changes as products evolve.

Regularly:

- Review misclassified queries
- Add new categories
- Merge overlapping categories
- Adjust keyword rules
- Retrain classifiers if needed

Continuous refinement improves both accuracy and cost efficiency.

---

# Example Production Flow

```
User
 │
 ▼
Classifier
 │
 ├──────── FAQ
 │          │
 │          ▼
 │     Static Answer
 │
 ├──────── Search
 │          │
 │          ▼
 │     Search API
 │
 ├──────── Docs
 │          │
 │          ▼
 │        RAG
 │
 ├──────── Coding
 │          │
 │          ▼
 │     Code Model
 │
 └──────── Agent
            │
            ▼
         Planning
         Tools
         Memory
         Reasoning
```

Only a subset of requests ever reaches the most expensive pipeline.

---

# Benefits

## Lower Cost

Simple requests avoid unnecessary LLM reasoning, reducing token consumption and infrastructure expenses.

## Reduced Latency

Static lookups and APIs respond much faster than complex LLM workflows, improving user experience.

## Higher Throughput

Efficient routing allows the same infrastructure to serve more users by reserving expensive pipelines for tasks that truly require them.

## Better Resource Utilization

Each pipeline is optimized for its specific workload instead of relying on a one-size-fits-all architecture.

## Improved Scalability

As traffic grows, specialized pipelines can be scaled independently, making the system easier to operate and more cost-effective.

---

# Best Practices

- Keep categories mutually exclusive where possible.
- Prefer deterministic routing (rules or keywords) for obvious intents.
- Use embeddings or small classifiers for ambiguous cases.
- Measure classification accuracy and routing latency.
- Log confidence scores and fallback decisions.
- Periodically audit misclassified queries.
- Continuously refine routing rules as user behavior evolves.

---

# Trade-offs

| Challenge | Description |
|-----------|-------------|
| Misclassification | Incorrect routing may send a query to the wrong pipeline, leading to poor answers or unnecessary costs. |
| Category Drift | User intents evolve over time, requiring updates to categories and routing logic. |
| Maintenance Overhead | Rules, classifiers, and routing policies need ongoing tuning and monitoring. |
| Cold Start | New products may lack sufficient historical data to design effective categories. |
| Additional Component | The classifier adds another service to deploy, monitor, and maintain, though its cost is usually far lower than unnecessary LLM inference. |

---

# Real-World Examples

### Customer Support Bots

- Password reset → FAQ
- Order status → Database lookup
- Return request → Workflow automation
- Technical troubleshooting → Agent

---

### Enterprise Knowledge Assistants

- HR policy → RAG
- Leave balance → HR API
- Payroll issue → Workflow
- Company handbook → Vector search

---

### Developer Copilots

- Explain error → Coding LLM
- Search documentation → RAG
- Run tests → Tool
- Refactor repository → Agent

---

### AI Productivity Assistants

- "What's on my calendar?" → Calendar API
- "Summarize this report." → RAG + LLM
- "Book a meeting." → Workflow automation
- "Plan my quarterly roadmap." → Agent

---

# Combining with Other Optimization Techniques

Query Classification becomes even more effective when combined with other inference optimization strategies:

| Technique | Benefit When Combined |
|----------|------------------------|
| Multi-Model Routing | Selects the appropriate model after choosing the correct pipeline. |
| Response Caching | FAQ and repeated queries can bypass inference entirely. |
| Semantic Caching | Similar queries reuse previous responses without reprocessing. |
| Prompt Caching | Reduces repeated processing of static prompts within each pipeline. |
| RAG | Invoked only for document-based questions instead of every request. |
| Tool-First Architecture | Deterministic tasks are solved using tools before any LLM reasoning. |
| Batch Processing | Non-urgent requests within a pipeline can be processed together for additional cost savings. |

---

# Research Papers

1. **BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding**  
   https://arxiv.org/abs/1810.04805

2. **Sentence-BERT: Sentence Embeddings using Siamese BERT Networks**  
   https://arxiv.org/abs/1908.10084

3. **DistilBERT, a distilled version of BERT**  
   https://arxiv.org/abs/1910.01108

4. **FastText: Bag of Tricks for Efficient Text Classification**  
   https://arxiv.org/abs/1607.01759

5. **Attention Is All You Need**  
   https://arxiv.org/abs/1706.03762

---

# Useful Articles & Blogs

### OpenAI

- https://platform.openai.com/docs/guides
- https://platform.openai.com/docs/guides/structured-outputs
- https://platform.openai.com/docs/guides/tools

### Anthropic

- https://docs.anthropic.com/en/docs/build-with-claude

### LangChain

- https://python.langchain.com/docs/tutorials/

### LlamaIndex

- https://docs.llamaindex.ai/

### Pinecone Learning Center

- https://www.pinecone.io/learn/

### Weaviate Blog

- https://weaviate.io/blog

### Hugging Face

- https://huggingface.co/docs

### Cohere

- https://docs.cohere.com/

### Sentence Transformers

- https://www.sbert.net/

---

# Summary

Query Classification is a foundational optimization technique that ensures each user request is handled by the **least expensive pipeline capable of producing a high-quality answer**. By separating FAQs, RAG queries, coding tasks, deterministic tool calls, and complex agent workflows, organizations can significantly reduce LLM costs, lower latency, improve scalability, and deliver faster, more reliable AI applications. When combined with caching, multi-model routing, RAG, and tool-first architectures, query classification forms a critical component of efficient, production-grade AI systems.