# Embedding Models Reference

> Last verified: January 2026

## Quick Selection Guide

| Use Case | Recommended Model | Dimensions | Price/1M |
|----------|-------------------|------------|----------|
| **General RAG** | Voyage 3-large | 1024 | $0.06 |
| **Budget RAG** | text-embedding-3-small | 1536 | $0.02 |
| **Code search** | Voyage Code 3 | 1024 | $0.06 |
| **Legal docs** | Voyage Law 2 | 1024 | $0.12 |
| **Finance** | Voyage Finance 2 | 1024 | $0.12 |
| **Free tier** | Gemini embedding-001 | 768-3072 | Free |
| **Self-hosted** | nomic-embed-text | 768 | Free |

## Model Comparison

| Model | Dimensions | Context | MTEB Score | Price/1M |
|-------|------------|---------|------------|----------|
| **Voyage 3-large** | 256-2048 | 32K | 68.3 | $0.06 |
| **Voyage 3.5** | 256-2048 | 32K | 67.1 | $0.06 |
| **Voyage 4** | 256-2048 | 32K | ~69 | $0.06 |
| **text-embedding-3-large** | 256-3072 | 8K | 64.6 | $0.13 |
| **text-embedding-3-small** | 512-1536 | 8K | 62.3 | $0.02 |
| **Gemini embedding-001** | 768-3072 | 2K | ~64 | Free |
| **Cohere embed-v3** | 1024 | 512 | 64.5 | $0.10 |

## OpenAI Embeddings

### Implementation

```python
from openai import OpenAI

client = OpenAI()

response = client.embeddings.create(
    model="text-embedding-3-large",
    input="Your text here",
    dimensions=1024,  # Optional: reduce dimensions
    encoding_format="float"  # or "base64"
)

embedding = response.data[0].embedding
print(f"Dimensions: {len(embedding)}")
```

### Batch Processing

```python
texts = ["First document", "Second document", "Third document"]

response = client.embeddings.create(
    model="text-embedding-3-small",
    input=texts
)

embeddings = [item.embedding for item in response.data]
```

### Available Models

| Model | Default Dims | Max Dims | Context | Price/1M |
|-------|-------------|----------|---------|----------|
| text-embedding-3-large | 3072 | 3072 | 8191 | $0.13 |
| text-embedding-3-small | 1536 | 1536 | 8191 | $0.02 |
| text-embedding-ada-002 | 1536 | 1536 | 8191 | $0.10 |

### Dimension Reduction

```python
# Reduce dimensions to save storage (Matryoshka training)
response = client.embeddings.create(
    model="text-embedding-3-large",
    input="Your text",
    dimensions=512  # Reduce from 3072
)
# Still useful for retrieval, lower storage cost
```

## Voyage AI Embeddings

### Implementation

```python
import voyageai

client = voyageai.Client()

result = client.embed(
    texts=["Your text here"],
    model="voyage-3-large",
    input_type="document",  # or "query"
    output_dimension=1024   # Optional
)

embedding = result.embeddings[0]
```

### Query vs Document

```python
# For indexing documents
doc_embeddings = client.embed(
    texts=documents,
    model="voyage-3-large",
    input_type="document"
)

# For search queries
query_embedding = client.embed(
    texts=[query],
    model="voyage-3-large",
    input_type="query"  # Optimized for retrieval
)
```

### Available Models

| Model | Best For | Dimensions | Context | Price/1M |
|-------|----------|------------|---------|----------|
| voyage-4-large | Highest quality | 256-2048 | 32K | $0.18 |
| voyage-4 | Balanced | 256-2048 | 32K | $0.10 |
| voyage-4-lite | Budget | 256-2048 | 32K | $0.02 |
| voyage-3-large | General | 256-2048 | 32K | $0.06 |
| voyage-3.5 | Fast | 256-2048 | 32K | $0.06 |
| voyage-code-3 | Code | 256-2048 | 32K | $0.06 |
| voyage-law-2 | Legal | 1024 | 16K | $0.12 |
| voyage-finance-2 | Finance | 1024 | 16K | $0.12 |

### Quantization Options

```python
# Reduce storage with quantization
result = client.embed(
    texts=["Your text"],
    model="voyage-3-large",
    output_dimension=512,
    output_dtype="int8"  # or "uint8", "binary", "ubinary"
)
# Binary: 1/32 storage, minimal quality loss
```

## Google Gemini Embeddings

### Implementation

```python
import google.generativeai as genai

genai.configure(api_key="YOUR_API_KEY")

result = genai.embed_content(
    model="models/text-embedding-004",
    content="Your text here",
    task_type="retrieval_document"
)

embedding = result['embedding']
```

### Task Types

```python
# Different embeddings for different tasks
TASK_TYPES = [
    "retrieval_query",      # Search queries
    "retrieval_document",   # Documents to search
    "semantic_similarity",  # Comparing texts
    "classification",       # Text classification
    "clustering"            # Grouping similar texts
]
```

### Available Models

| Model | Dimensions | Context | Price |
|-------|------------|---------|-------|
| text-embedding-004 | 768 | 2048 | Free |
| embedding-001 | 768 | 2048 | Free |

**Generous free tier** makes Gemini good for development/testing.

## Cohere Embeddings

### Implementation

```python
import cohere

co = cohere.Client("YOUR_API_KEY")

response = co.embed(
    texts=["Your text here"],
    model="embed-english-v3.0",
    input_type="search_document"  # or "search_query"
)

embeddings = response.embeddings
```

### Available Models

| Model | Dimensions | Languages | Price/1M |
|-------|------------|-----------|----------|
| embed-english-v3.0 | 1024 | English | $0.10 |
| embed-multilingual-v3.0 | 1024 | 100+ | $0.10 |

## Self-Hosted (Ollama)

### Setup

```bash
# Install Ollama and pull model
ollama pull nomic-embed-text

# Or for multilingual
ollama pull mxbai-embed-large
```

### Implementation

```python
import ollama

response = ollama.embeddings(
    model="nomic-embed-text",
    prompt="Your text here"
)

embedding = response["embedding"]
```

### Available Models

| Model | Dimensions | Context | Size |
|-------|------------|---------|------|
| nomic-embed-text | 768 | 8192 | 274MB |
| mxbai-embed-large | 1024 | 512 | 670MB |
| all-minilm | 384 | 256 | 45MB |

## Best Practices

### 1. Chunking Strategy

```python
# Optimal chunk sizes by model
CHUNK_SIZES = {
    "voyage-3-large": 512,      # Can handle 32K but 512 is optimal
    "text-embedding-3-large": 512,
    "text-embedding-3-small": 256,
    "nomic-embed-text": 512,
}

# Overlap for continuity
OVERLAP = 50  # tokens
```

### 2. Batch for Efficiency

```python
# Process in batches to reduce API calls
BATCH_SIZE = 100  # Voyage supports up to 128

for i in range(0, len(texts), BATCH_SIZE):
    batch = texts[i:i + BATCH_SIZE]
    embeddings = client.embed(texts=batch, model="voyage-3-large")
```

### 3. Dimension Selection

```python
# Balance quality vs storage
DIMENSION_GUIDE = {
    "high_quality": 1024,    # Best retrieval
    "balanced": 512,         # Good tradeoff
    "storage_limited": 256,  # Minimal loss
}
```

### 4. Input Type Matters

```python
# Always specify for retrieval tasks
# Query embeddings and document embeddings are asymmetric
doc_emb = embed(docs, input_type="document")
query_emb = embed(query, input_type="query")  # Different!
```

## Common Gotchas

1. **Token limits vary widely**
   - OpenAI: 8K tokens
   - Voyage: 32K tokens
   - Gemini: 2K tokens

2. **Dimension reduction quality**
   - Models with Matryoshka training (OpenAI, Voyage) handle it well
   - Others may degrade significantly

3. **Query vs Document embeddings**
   - Many models optimize differently for each
   - Always use the correct input type

4. **Batch limits**
   - OpenAI: No explicit limit but watch rate limits
   - Voyage: 128 texts, 1M tokens per request

5. **Caching embeddings**
   - Store embeddings to avoid recomputing
   - Include model version in cache key

6. **Cosine vs dot product**
   - Most models optimize for cosine similarity
   - Normalize vectors if using dot product
