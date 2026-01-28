# RAG (Retrieval-Augmented Generation) Guide

## Quick Stack Recommendations

| Use Case | Embedding | LLM | Vector DB |
|----------|-----------|-----|-----------|
| **General knowledge base** | Voyage 3-large | Claude Sonnet 4.5 | Pinecone/Qdrant |
| **Code documentation** | Voyage Code 3 | Claude Sonnet 4.5 | Pinecone |
| **Legal documents** | Voyage Law 2 | Claude Opus 4.5 | Weaviate |
| **Budget/startup** | text-embedding-3-small | Claude Haiku 4.5 | Chroma |
| **Self-hosted** | nomic-embed-text | Llama 3.3 | Milvus |
| **Free tier testing** | Gemini embedding | Gemini 2.5 Flash | Chroma |

## Embedding Selection

### By Domain

| Domain | Model | Why |
|--------|-------|-----|
| **General** | Voyage 3-large | Best MTEB scores |
| **Code** | Voyage Code 3 | 13.8% better than OpenAI on code |
| **Legal** | Voyage Law 2 | Domain-optimized |
| **Finance** | Voyage Finance 2 | Domain-optimized |
| **Multilingual** | Voyage 3-large | Strong on 100+ languages |
| **Budget** | text-embedding-3-small | $0.02/1M, good enough |

### Configuration

```python
# Voyage - best quality
VOYAGE_CONFIG = {
    "model": "voyage-3-large",
    "input_type": "document",  # or "query" for search
    "output_dimension": 1024,  # Can reduce for storage
}

# OpenAI - widely available
OPENAI_CONFIG = {
    "model": "text-embedding-3-large",
    "dimensions": 1024,  # Reduce from 3072
}

# Budget option
BUDGET_CONFIG = {
    "model": "text-embedding-3-small",
    "dimensions": 512,  # Reduce from 1536
}
```

## Chunking Strategy

### Chunk Sizes by Content Type

| Content Type | Chunk Size | Overlap | Notes |
|--------------|------------|---------|-------|
| **Documentation** | 512 tokens | 50 | Preserve sections |
| **Code** | 256-512 | 0 | Chunk by function/class |
| **Legal** | 1024 | 100 | Preserve paragraph context |
| **Chat logs** | 256 | 0 | Per message/exchange |
| **Research papers** | 512 | 50 | Preserve citations |

### Implementation

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# General purpose
splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,
    chunk_overlap=50,
    separators=["\n\n", "\n", ". ", " ", ""]
)

# For code
code_splitter = RecursiveCharacterTextSplitter.from_language(
    language="python",
    chunk_size=500,
    chunk_overlap=0
)

# For markdown
md_splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,
    chunk_overlap=50,
    separators=["\n## ", "\n### ", "\n\n", "\n", " "]
)
```

## LLM Selection for RAG

### By Task

| Task | Model | Config |
|------|-------|--------|
| **Q&A** | Claude Sonnet 4.5 | temp=0 |
| **Summarization** | Claude Haiku 4.5 | temp=0.3 |
| **Analysis** | Claude Opus 4.5 | thinking enabled |
| **High volume** | Claude Haiku 4.5 | temp=0 |
| **Complex reasoning** | Claude Opus 4.5 | effort=high |

### RAG Prompt Template

```python
RAG_PROMPT = """Answer the question based on the provided context.

Context:
{context}

Question: {question}

Instructions:
- Only use information from the context
- If the context doesn't contain the answer, say so
- Cite specific sections when possible
- Be concise but complete

Answer:"""
```

### Citation-Aware Prompt

```python
CITATION_PROMPT = """Answer with citations from the provided documents.

Documents:
{documents}

Question: {question}

Format your response as:
[Your answer here]

Sources:
- [1] Document title, relevant quote
- [2] Document title, relevant quote

If no documents support your answer, state that clearly."""
```

## Complete RAG Pipeline

### Setup

```python
import voyageai
import anthropic
from qdrant_client import QdrantClient

# Initialize clients
voyage = voyageai.Client()
claude = anthropic.Anthropic()
qdrant = QdrantClient(url="http://localhost:6333")

COLLECTION = "knowledge_base"
EMBEDDING_MODEL = "voyage-3-large"
LLM_MODEL = "claude-sonnet-4-5-20250929"
```

### Indexing

```python
def index_documents(documents: list[dict]):
    """Index documents with metadata."""
    
    # Extract text for embedding
    texts = [doc["content"] for doc in documents]
    
    # Generate embeddings
    result = voyage.embed(
        texts=texts,
        model=EMBEDDING_MODEL,
        input_type="document"
    )
    
    # Upsert to vector DB
    points = [
        {
            "id": i,
            "vector": result.embeddings[i],
            "payload": {
                "content": doc["content"],
                "title": doc.get("title", ""),
                "source": doc.get("source", ""),
                "metadata": doc.get("metadata", {})
            }
        }
        for i, doc in enumerate(documents)
    ]
    
    qdrant.upsert(collection_name=COLLECTION, points=points)
```

### Retrieval

```python
def retrieve(query: str, top_k: int = 5) -> list[dict]:
    """Retrieve relevant documents."""
    
    # Embed query (note: input_type="query")
    query_embedding = voyage.embed(
        texts=[query],
        model=EMBEDDING_MODEL,
        input_type="query"
    ).embeddings[0]
    
    # Search
    results = qdrant.search(
        collection_name=COLLECTION,
        query_vector=query_embedding,
        limit=top_k
    )
    
    return [
        {
            "content": hit.payload["content"],
            "title": hit.payload.get("title", ""),
            "score": hit.score
        }
        for hit in results
    ]
```

### Generation

```python
def generate_answer(query: str, context_docs: list[dict]) -> str:
    """Generate answer from retrieved context."""
    
    # Format context
    context = "\n\n---\n\n".join([
        f"[{i+1}] {doc['title']}\n{doc['content']}"
        for i, doc in enumerate(context_docs)
    ])
    
    prompt = RAG_PROMPT.format(context=context, question=query)
    
    response = claude.messages.create(
        model=LLM_MODEL,
        max_tokens=2000,
        temperature=0,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.content[0].text
```

### Complete Query

```python
def rag_query(query: str) -> str:
    """Full RAG pipeline."""
    
    # 1. Retrieve
    docs = retrieve(query, top_k=5)
    
    # 2. Filter by relevance score
    relevant_docs = [d for d in docs if d["score"] > 0.7]
    
    if not relevant_docs:
        return "I couldn't find relevant information to answer this question."
    
    # 3. Generate
    answer = generate_answer(query, relevant_docs)
    
    return answer
```

## Advanced Patterns

### Hybrid Search (Dense + Sparse)

```python
from qdrant_client.models import models

def hybrid_search(query: str, top_k: int = 5):
    # Dense embedding
    dense_vector = voyage.embed(
        texts=[query],
        model="voyage-3-large",
        input_type="query"
    ).embeddings[0]
    
    # Sparse (BM25-style) - if supported by your vector DB
    results = qdrant.search(
        collection_name=COLLECTION,
        query_vector=dense_vector,
        query_filter=models.Filter(
            must=[
                models.FieldCondition(
                    key="content",
                    match=models.MatchText(text=query)  # Keyword match
                )
            ]
        ),
        limit=top_k
    )
    
    return results
```

### Reranking

```python
import cohere

co = cohere.Client()

def retrieve_and_rerank(query: str, top_k: int = 5):
    # Get more candidates than needed
    candidates = retrieve(query, top_k=20)
    
    # Rerank
    reranked = co.rerank(
        model="rerank-english-v3.0",
        query=query,
        documents=[c["content"] for c in candidates],
        top_n=top_k
    )
    
    # Return top reranked
    return [candidates[r.index] for r in reranked.results]
```

### Query Expansion

```python
def expand_query(query: str) -> list[str]:
    """Generate multiple query variations."""
    
    response = claude.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=500,
        messages=[{
            "role": "user",
            "content": f"""Generate 3 alternative phrasings of this search query.
Return only the queries, one per line.

Query: {query}"""
        }]
    )
    
    variations = response.content[0].text.strip().split("\n")
    return [query] + variations[:3]

def multi_query_retrieve(query: str, top_k: int = 5):
    """Retrieve using multiple query variations."""
    
    queries = expand_query(query)
    all_results = []
    seen_ids = set()
    
    for q in queries:
        results = retrieve(q, top_k=top_k)
        for r in results:
            if r["id"] not in seen_ids:
                all_results.append(r)
                seen_ids.add(r["id"])
    
    # Sort by score and return top_k
    return sorted(all_results, key=lambda x: x["score"], reverse=True)[:top_k]
```

## Cost Optimization

### 1. Embedding Caching

```python
import hashlib
import redis

cache = redis.Redis()

def get_embedding(text: str) -> list[float]:
    # Check cache
    key = hashlib.md5(text.encode()).hexdigest()
    cached = cache.get(f"emb:{key}")
    
    if cached:
        return json.loads(cached)
    
    # Generate and cache
    embedding = voyage.embed(texts=[text], model="voyage-3-large").embeddings[0]
    cache.setex(f"emb:{key}", 86400, json.dumps(embedding))  # 24h TTL
    
    return embedding
```

### 2. Dimension Reduction

```python
# Reduce dimensions for storage savings
DIMENSION_CONFIGS = {
    "high_quality": 1024,   # Best retrieval
    "balanced": 512,        # Good tradeoff
    "storage_optimized": 256  # 4x smaller, slight quality loss
}

embedding = voyage.embed(
    texts=[text],
    model="voyage-3-large",
    output_dimension=DIMENSION_CONFIGS["balanced"]
).embeddings[0]
```

### 3. Tiered Retrieval

```python
def tiered_rag(query: str):
    """Use cheaper model first, upgrade if needed."""
    
    # Fast retrieval with Haiku
    docs = retrieve(query, top_k=3)
    
    if all(d["score"] > 0.85 for d in docs):
        # High confidence - use Haiku
        return generate_answer(query, docs, model="claude-haiku-4-5-20251001")
    else:
        # Lower confidence - use Sonnet with more docs
        docs = retrieve(query, top_k=7)
        return generate_answer(query, docs, model="claude-sonnet-4-5-20250929")
```
