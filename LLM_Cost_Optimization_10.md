# 10. Structured Outputs

## Core Idea

**Structured Outputs** is an LLM optimization technique where the model is constrained to generate responses that strictly follow a predefined schema (such as JSON, XML, or a typed object) instead of free-form natural language.

Rather than asking the model to "format the response correctly," the API enforces a structure, significantly reducing parsing failures and eliminating unnecessary verbose text.

This optimization improves:

- Lower token usage
- Faster downstream processing
- More reliable integrations
- Fewer parsing errors
- Reduced retry costs
- Better automation reliability

---

# Why Structured Outputs?

Many production LLM applications require machine-readable responses.

Without structured outputs, models may produce:

- Extra explanations
- Markdown formatting
- Missing fields
- Invalid JSON
- Hallucinated keys
- Incorrect data types

Example:

Instead of returning:

> The user's sentiment appears to be positive with a confidence of around 92%.

You can receive:

```json
{
  "sentiment": "positive",
  "confidence": 0.92
}
```

This removes unnecessary output tokens while making the response immediately consumable by downstream applications.

---

# How Structured Outputs Work

Instead of prompting:

> Return the answer as JSON.

Modern APIs allow developers to provide an explicit schema.

The model is then constrained to generate output matching that schema.

Typical workflow:

```
User Query
      │
      ▼
LLM API
      │
Schema Provided
      │
      ▼
Model Generates Structured Response
      │
Validation
      │
      ▼
Application Logic
```

---

# Implementation Steps

## 1. Define a Strict Schema

Use one of the following:

- JSON Schema
- Pydantic Model
- Dataclass
- Typed Object
- XML Schema (less common)

Example (Pydantic):

```python
from pydantic import BaseModel

class WeatherResponse(BaseModel):
    city: str
    temperature: float
    condition: str
```

---

## 2. Use Structured Output APIs

Most modern LLM providers support:

- JSON mode
- Structured Outputs
- Tool Calling
- Function Calling

Instead of relying on prompt engineering alone, pass the schema directly to the model.

---

## 3. Validate the Output

Always validate before using.

Validation catches:

- Missing fields
- Wrong data types
- Invalid enums
- Unexpected objects

Example:

```python
WeatherResponse.model_validate(response_json)
```

Only retry when validation actually fails.

---

## 4. Keep Responses Purely Structured

Avoid prompts like:

> Explain your reasoning and then return JSON.

Instead:

```
Return ONLY valid JSON.
```

or use native structured output support.

Mixing explanations with structured data increases output tokens and often breaks parsing.

---

# Example Without Structured Output

Prompt:

```
Extract invoice details.
```

Possible output:

```
The invoice number appears to be INV-1045.
The vendor is ABC Pvt Ltd.
The total amount is ₹12,500.
```

Problems:

- Extra text
- Parsing required
- Regex needed
- Fragile automation

---

# Example With Structured Output

Schema:

```json
{
  "invoice_number": "string",
  "vendor": "string",
  "amount": "number"
}
```

Output:

```json
{
  "invoice_number": "INV-1045",
  "vendor": "ABC Pvt Ltd",
  "amount": 12500
}
```

Immediately usable.

---

# Token Savings

Verbose response:

```
The customer's sentiment is positive.
I estimate the confidence to be approximately 94%.
```

≈ 25–35 output tokens

Structured response:

```json
{
  "sentiment": "positive",
  "confidence": 0.94
}
```

≈ 10–15 output tokens

Across millions of requests, these savings become substantial.

---

# Benefits

## Lower Output Tokens

Only the required fields are generated.

No unnecessary explanations.

---

## Reduced Parsing Errors

Applications no longer depend on:

- Regex
- String matching
- Prompt-specific formatting

---

## Fewer Retries

Malformed outputs are much less common.

Retries happen only when schema validation fails.

---

## Better Reliability

Responses are deterministic and easier to consume.

Ideal for production APIs.

---

## Easier Backend Integration

Structured responses map directly into:

- Databases
- APIs
- Workflows
- Event systems
- Agent pipelines

---

# Common Use Cases

## Information Extraction

Example:

```json
{
  "name": "...",
  "email": "...",
  "phone": "..."
}
```

---

## Invoice Processing

```json
{
  "invoice_id": "...",
  "vendor": "...",
  "total": 15230
}
```

---

## Document Classification

```json
{
  "category": "Insurance",
  "confidence": 0.98
}
```

---

## Agent Actions

```json
{
  "action": "search_database",
  "arguments": {
    "customer_id": 1024
  }
}
```

---

## Event Logging

```json
{
  "event": "invoice_validated",
  "status": "success",
  "latency_ms": 281
}
```

---

# Best Practices

- Define explicit schemas.
- Use native structured-output APIs whenever available.
- Validate every response before using it.
- Retry only on validation failures.
- Keep schemas small and focused.
- Avoid mixing explanations with structured data.
- Prefer enums for fixed-value fields.
- Use nullable fields only when necessary.
- Version schemas if they evolve over time.
- Log validation failures for monitoring and debugging.

---

# Trade-offs

| Advantage | Limitation |
|-----------|------------|
| Lower token usage | Less natural-language flexibility |
| Reliable parsing | Schema mismatches can still occur |
| Fewer retries | Requires schema maintenance |
| Easier backend integration | Complex nested schemas can increase development effort |
| Better automation | Not ideal when free-form narrative responses are required |

---

# When to Use Structured Outputs

Use Structured Outputs when your application needs:

- Information extraction
- API responses
- Function calling
- Agent workflows
- Database insertion
- Event logging
- Invoice processing
- Classification
- Decision support systems
- Automation pipelines

Avoid them when users primarily expect rich, conversational, or creative text.

---

# Provider Support

| Provider | Feature |
|----------|---------|
| OpenAI | Structured Outputs, JSON Mode, Function Calling |
| Anthropic | Tool Use |
| Google Gemini | Function Calling, JSON Responses |
| Groq | Tool Calling (model dependent) |
| Mistral AI | Function Calling |
| Cohere | Tool Use |

---

# Real-World Example

Suppose you're building an invoice processing pipeline.

Instead of asking:

> Extract the invoice details.

Define a schema:

```json
{
  "invoice_number": "string",
  "vendor_name": "string",
  "invoice_date": "string",
  "currency": "string",
  "total_amount": "number"
}
```

The model returns only the required fields, which are validated and directly inserted into your database or passed to downstream services. This avoids custom parsing logic, reduces output tokens, and minimizes retries caused by malformed responses.

---

# Summary

Structured Outputs improve both **cost efficiency** and **reliability** by constraining LLM responses to predefined schemas. They reduce unnecessary output tokens, eliminate most parsing errors, and make integrations with APIs, databases, and agent workflows significantly more robust.

For production-grade LLM systems, structured outputs are considered a best practice whenever responses are intended for machine consumption rather than human reading.

---

# References

## Research Papers

1. OpenAI. **Introducing Structured Outputs in the API** (2024)  
   https://openai.com/index/introducing-structured-outputs/

2. OpenAI API Documentation – Structured Outputs  
   https://platform.openai.com/docs/guides/structured-outputs

3. OpenAI API Documentation – Function Calling  
   https://platform.openai.com/docs/guides/function-calling

4. JSON Schema Specification  
   https://json-schema.org/

5. Pydantic Documentation  
   https://docs.pydantic.dev/latest/

---

## Articles & Blogsv

- OpenAI Cookbook – Structured Outputs Examples  
  https://cookbook.openai.com/

- Anthropic Documentation – Tool Use  
  https://docs.anthropic.com/

- Google Gemini Function Calling Documentation  
  https://ai.google.dev/

- Microsoft Learn – JSON Schema Fundamentals  
  https://learn.microsoft.com/

- Pydantic Documentation  
  https://docs.pydantic.dev/latest/

- JSON Schema Official Documentation  
  https://json-schema.org/learn/

- Martin Fowler – Schema Design & API Reliability  
  https://martinfowler.com/

---

# Key Takeaways

- Constrain model responses with predefined schemas.
- Use native structured-output or function-calling APIs instead of prompt-only formatting.
- Validate outputs before downstream processing.
- Retry only when schema validation fails.
- Keep schemas focused and avoid mixing prose with structured data.
- Structured outputs reduce token costs, parsing errors, and integration complexity, making them ideal for production LLM applications.