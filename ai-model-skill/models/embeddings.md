# Embedding Models

## Voyage AI (Recommended for Quality)

### voyage-3-large
**Best overall quality for general-purpose embeddings**
- **Dimensions**: 256 - 2,048 (configurable via Matryoshka)
- **Context Window**: 32,000 tokens
- **Pricing**: $0.06 / 1M tokens
- **Free Tier**: 200M tokens free

State-of-the-art on MTEB, outperforms OpenAI by ~10%

### voyage-3.5 / voyage-3.5-lite
**Latest Voyage models (2025)**
- **voyage-3.5**: $0.06 / 1M tokens, best quality
- **voyage-3.5-lite**: $0.02 / 1M tokens, best value
- **Context**: 32,000 tokens
- **Features**: Matryoshka embeddings, int8/binary quantization

Outperforms OpenAI-v3-large by 6-8%

### voyage-code-3
**Best for code retrieval**
- **Dimensions**: Up to 2,048
- **Context Window**: 32,000 tokens
- **Pricing**: $0.06 / 1M tokens
- **Free Tier**: 200M tokens free

Outperforms OpenAI and CodeSage by 13-17%

### voyage-law-2
**Optimized for legal domain**
- **Pricing**: ~$0.12 / 1M tokens
- **Free Tier**: 50M tokens free

### voyage-finance-2
**Optimized for financial domain**
- **Pricing**: ~$0.12 / 1M tokens
- **Free Tier**: 50M tokens free

---

## OpenAI Embeddings

### text-embedding-3-large
**Highest quality OpenAI embedding**
- **Dimensions**: 256 - 3,072 (configurable)
- **Context Window**: 8,191 tokens
- **Pricing**: $0.13 / 1M tokens
- **Batch API**: $0.065 / 1M tokens (50% off)

### text-embedding-3-small
**Best value OpenAI embedding**
- **Dimensions**: 512 - 1,536 (configurable)
- **Context Window**: 8,191 tokens
- **Pricing**: $0.02 / 1M tokens
- **Batch API**: $0.01 / 1M tokens

Excellent for budget-conscious production use

### text-embedding-ada-002 (Legacy)
- **Dimensions**: 1,536 (fixed)
- **Pricing**: $0.10 / 1M tokens
- **Recommendation**: Use text-embedding-3-small instead

---

## Google Gemini Embeddings

### gemini-embedding-001
**Free option with good quality**
- **Dimensions**: 768 - 3,072 (configurable)
- **Context Window**: Large
- **Pricing**: Free tier available!

Good for prototyping and budget applications

---

## Cohere Embeddings

### embed-v4 / embed-english-v3
- **Dimensions**: Up to 1,024
- **Pricing**: ~$0.10 / 1M tokens
- Good multilingual support

---

## Quick Comparison

| Model | Quality | Price/1M | Dimensions | Context |
|-------|---------|----------|------------|---------|
| voyage-3-large | ⭐⭐⭐⭐⭐ | $0.06 | 256-2048 | 32K |
| voyage-3.5 | ⭐⭐⭐⭐⭐ | $0.06 | 256-2048 | 32K |
| voyage-3.5-lite | ⭐⭐⭐⭐ | $0.02 | 256-2048 | 32K |
| voyage-code-3 | ⭐⭐⭐⭐⭐ | $0.06 | 256-2048 | 32K |
| text-embedding-3-large | ⭐⭐⭐⭐ | $0.13 | 256-3072 | 8K |
| text-embedding-3-small | ⭐⭐⭐ | $0.02 | 512-1536 | 8K |
| gemini-embedding-001 | ⭐⭐⭐ | Free | 768-3072 | Large |

---

## Selection Guide

| Use Case | Recommended | Reason |
|----------|-------------|--------|
| General RAG (quality) | voyage-3.5 | Best MTEB scores |
| General RAG (budget) | text-embedding-3-small | $0.02/1M, good quality |
| Code search | voyage-code-3 | Optimized for code |
| Legal documents | voyage-law-2 | Domain-specific |
| Financial docs | voyage-finance-2 | Domain-specific |
| Free prototyping | gemini-embedding-001 | No cost |
| High-volume prod | voyage-3.5-lite | $0.02/1M, great quality |

---

## Code Examples

### Voyage AI
```javascript
import Anthropic from '@anthropic-ai/sdk';
// Or use Voyage SDK directly

// Using Voyage API directly
const response = await fetch('https://api.voyageai.com/v1/embeddings', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${VOYAGE_API_KEY}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    model: 'voyage-3-large',
    input: ['Hello world', 'Another text'],
    input_type: 'document',  // or 'query' for search queries
    output_dimension: 1024,   // Matryoshka: choose dimension
    output_dtype: 'float'     // or 'int8', 'binary'
  })
});
```

### OpenAI
```javascript
import OpenAI from 'openai';

const openai = new OpenAI();

const response = await openai.embeddings.create({
  model: 'text-embedding-3-small',
  input: ['Hello world', 'Another text'],
  dimensions: 1024  // Optional: reduce dimensions
});

const embeddings = response.data.map(d => d.embedding);
```

### Google Gemini
```javascript
import { GoogleGenerativeAI } from "@google/generative-ai";

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
const model = genAI.getGenerativeModel({ model: "gemini-embedding-001" });

const result = await model.embedContent("Hello world");
const embedding = result.embedding.values;
```

---

## Dimension Reduction (Matryoshka)

Modern embeddings support flexible dimensions:

```javascript
// Voyage: specify output_dimension
{
  model: 'voyage-3-large',
  output_dimension: 512  // Reduce from 2048 to 512
}

// OpenAI: specify dimensions
{
  model: 'text-embedding-3-large',
  dimensions: 1024  // Reduce from 3072 to 1024
}
```

**Trade-offs:**
- Lower dimensions = smaller storage, faster search
- Quality degrades gracefully (not linearly)
- Test with your data to find optimal dimension

---

## Quantization Options (Voyage)

Reduce storage costs further:

```javascript
{
  model: 'voyage-3.5',
  output_dtype: 'int8'    // 4x smaller than float32
  // or 'binary'          // 32x smaller, some quality loss
}
```

**Storage savings:**
- float32 (1024 dims): 4KB per vector
- int8 (1024 dims): 1KB per vector
- binary (1024 dims): 128 bytes per vector

---

## Cost Optimization Tips

1. **Use batch processing** - OpenAI Batch API gives 50% off
2. **Reduce dimensions** - 512-1024 often sufficient
3. **Use quantization** - int8 for minimal quality loss
4. **Cache embeddings** - Don't re-embed unchanged documents
5. **Use appropriate model** - Don't use voyage-3-large for simple tasks

---

## Performance Notes

- **Voyage** models generally outperform OpenAI on retrieval benchmarks
- **voyage-code-3** significantly better for code than general-purpose models
- **Domain-specific** models (law, finance) worth the premium for those domains
- **Gemini** embeddings are good value for free tier users
