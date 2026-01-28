# RAG Pipeline Guide

## Embedding Selection

### General Purpose
| Model | Quality | Price/1M | Dimensions | Context |
|-------|---------|----------|------------|---------|
| voyage-3.5 | ⭐⭐⭐⭐⭐ | $0.06 | 256-2048 | 32K |
| voyage-3-large | ⭐⭐⭐⭐⭐ | $0.06 | 256-2048 | 32K |
| text-embedding-3-large | ⭐⭐⭐⭐ | $0.13 | 256-3072 | 8K |
| text-embedding-3-small | ⭐⭐⭐ | $0.02 | 512-1536 | 8K |
| gemini-embedding-001 | ⭐⭐⭐ | Free | 768-3072 | Large |

### Code-Specific
| Model | Quality | Price/1M | Best For |
|-------|---------|----------|----------|
| voyage-code-3 | ⭐⭐⭐⭐⭐ | $0.06 | Code search, docs |

### Domain-Specific
| Model | Domain | Price/1M |
|-------|--------|----------|
| voyage-law-2 | Legal | ~$0.12 |
| voyage-finance-2 | Finance | ~$0.12 |

---

## LLM Selection for RAG

### Question Answering
| Model | Best For | Price |
|-------|----------|-------|
| Claude Sonnet 4.5 | Accurate answers, citations | $3/$15 |
| Gemini 2.5 Flash | Fast responses, long context | $0.30/$2.50 |
| Claude Haiku 4.5 | High-volume Q&A | $1/$5 |

### Document Analysis
| Model | Best For | Price |
|-------|----------|-------|
| Claude Opus 4.5 | Deep analysis | $5/$25 |
| GPT-5.2 | Complex reasoning | $1.75/$14 |
| Gemini 2.5 Pro | Very long docs (1M) | $1.25/$10 |

### High-Volume/Budget
| Model | Best For | Price |
|-------|----------|-------|
| DeepSeek-chat | Budget RAG | $0.28/$0.42 |
| Gemini 2.5 Flash-Lite | Cheapest | $0.10/$0.40 |

---

## Complete RAG Pipeline

### 1. Indexing Phase

```javascript
// Using Voyage AI (best quality)
async function embedDocuments(documents) {
  const response = await fetch('https://api.voyageai.com/v1/embeddings', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${VOYAGE_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      model: 'voyage-3.5',
      input: documents.map(d => d.text),
      input_type: 'document',
      output_dimension: 1024  // Reduce for storage savings
    })
  });
  
  const data = await response.json();
  return data.embeddings;
}
```

```javascript
// Using OpenAI (alternative)
async function embedDocuments(documents) {
  const response = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: documents.map(d => d.text),
    dimensions: 1024
  });
  
  return response.data.map(d => d.embedding);
}
```

### 2. Query Phase

```javascript
async function embedQuery(query) {
  const response = await fetch('https://api.voyageai.com/v1/embeddings', {
    body: JSON.stringify({
      model: 'voyage-3.5',
      input: [query],
      input_type: 'query'  // Important: specify query type
    })
  });
  
  return response.json();
}
```

### 3. Retrieval Phase

```javascript
async function retrieveRelevant(queryEmbedding, vectorStore, topK = 5) {
  // Using your vector database (Pinecone, Weaviate, pgvector, etc.)
  const results = await vectorStore.query({
    vector: queryEmbedding,
    topK: topK,
    includeMetadata: true
  });
  
  return results.matches;
}
```

### 4. Generation Phase

```javascript
async function generateAnswer(query, context) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-5-20250929',
    max_tokens: 2000,
    system: `You are a helpful assistant. Answer based on the provided context.
    If the answer isn't in the context, say so.`,
    messages: [{
      role: 'user',
      content: `Context:
${context.map(c => c.text).join('\n\n')}

Question: ${query}`
    }]
  });
  
  return response.content[0].text;
}
```

---

## Advanced Patterns

### Hybrid Search (Vector + Keyword)
```javascript
async function hybridSearch(query, vectorStore) {
  // Vector search
  const vectorResults = await vectorStore.query({
    vector: await embedQuery(query),
    topK: 10
  });
  
  // Keyword search (BM25)
  const keywordResults = await keywordStore.search(query, { topK: 10 });
  
  // Combine with RRF (Reciprocal Rank Fusion)
  return reciprocalRankFusion([vectorResults, keywordResults]);
}
```

### Reranking
```javascript
// Using Voyage reranker
async function rerank(query, documents) {
  const response = await fetch('https://api.voyageai.com/v1/rerank', {
    body: JSON.stringify({
      model: 'rerank-2.5',
      query: query,
      documents: documents.map(d => d.text),
      top_k: 5
    })
  });
  
  return response.json();
}
```

### Query Expansion
```javascript
async function expandQuery(originalQuery) {
  const response = await anthropic.messages.create({
    model: 'claude-haiku-4-5-20251001',  // Fast, cheap
    max_tokens: 200,
    messages: [{
      role: 'user',
      content: `Generate 3 alternative phrasings of this search query:
"${originalQuery}"

Return only the queries, one per line.`
    }]
  });
  
  const variations = response.content[0].text.split('\n');
  return [originalQuery, ...variations];
}
```

---

## Chunking Strategies

### Fixed Size
```javascript
function fixedSizeChunk(text, chunkSize = 512, overlap = 50) {
  const chunks = [];
  for (let i = 0; i < text.length; i += chunkSize - overlap) {
    chunks.push(text.slice(i, i + chunkSize));
  }
  return chunks;
}
```

### Semantic Chunking
```javascript
// Use sentence boundaries + embedding similarity
function semanticChunk(text, maxTokens = 512) {
  const sentences = text.split(/[.!?]+/);
  const chunks = [];
  let currentChunk = [];
  let currentTokens = 0;
  
  for (const sentence of sentences) {
    const sentenceTokens = estimateTokens(sentence);
    if (currentTokens + sentenceTokens > maxTokens) {
      chunks.push(currentChunk.join(' '));
      currentChunk = [sentence];
      currentTokens = sentenceTokens;
    } else {
      currentChunk.push(sentence);
      currentTokens += sentenceTokens;
    }
  }
  
  if (currentChunk.length) chunks.push(currentChunk.join(' '));
  return chunks;
}
```

---

## Cost Optimization

### Embedding Costs
```
10,000 documents × 500 tokens = 5M tokens

voyage-3.5:              $0.30 (5M × $0.06/1M)
text-embedding-3-small:  $0.10 (5M × $0.02/1M)
gemini-embedding-001:    $0.00 (free tier)
```

### Generation Costs (per 1000 queries)
```
Assuming 2000 context tokens + 500 output tokens per query:

Claude Sonnet 4.5: ~$37.50 (2M × $3 + 0.5M × $15)
Claude Haiku 4.5:  ~$4.50  (2M × $1 + 0.5M × $5)
DeepSeek:          ~$0.77  (2M × $0.28 + 0.5M × $0.42)
```

### Storage Optimization
```javascript
// Reduce dimensions to save vector storage
{
  model: 'voyage-3.5',
  output_dimension: 512  // vs default 2048
}

// Use quantization
{
  model: 'voyage-3.5',
  output_dtype: 'int8'  // 4x smaller than float32
}
```

---

## Model Selection Matrix

| Scenario | Embeddings | LLM | Reason |
|----------|------------|-----|--------|
| Quality focus | voyage-3.5 | Sonnet 4.5 | Best retrieval + generation |
| Budget | text-embedding-3-small | DeepSeek | Cheapest viable |
| High volume | voyage-3.5-lite | Haiku 4.5 | Fast + cheap |
| Long docs | voyage-3.5 | Gemini 2.5 Pro | 1M context |
| Code RAG | voyage-code-3 | Sonnet 4.5 | Optimized for code |
| Free tier | gemini-embedding | Gemini Flash | No cost |
