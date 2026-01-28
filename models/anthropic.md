# Anthropic Claude Models

## Claude 4.5 Family (Latest - November 2025)

### Claude Opus 4.5
**Model String**: `claude-opus-4-5-20251101` or `claude-opus-4.5`
- **Context Window**: 200,000 tokens
- **Max Output**: 64,000 tokens
- **Knowledge Cutoff**: May 2025
- **Pricing**: $5.00 / $25.00 per 1M tokens (input/output)
- **Cache Write (5m)**: $6.25 / MTok
- **Cache Write (1h)**: $10.00 / MTok
- **Cache Read**: $0.50 / MTok
- **Batch API**: $2.50 / $12.50 per 1M (50% off)

**Unique Feature - Effort Parameter:**
```javascript
{
  "model": "claude-opus-4-5-20251101",
  "effort": "low" | "medium" | "high"  // Only Opus 4.5 supports this
}
```

Best for: Most complex tasks, deep reasoning, long-horizon coding, multi-agent coordination

**SWE-bench Verified**: 80.9% (state-of-the-art)

---

### Claude Sonnet 4.5
**Model String**: `claude-sonnet-4-5-20250929` or `claude-sonnet-4.5`
- **Context Window**: 200,000 tokens (1M beta available for tier 4)
- **Max Output**: 64,000 tokens
- **Knowledge Cutoff**: January 2025
- **Pricing**: $3.00 / $15.00 per 1M tokens

**Long Context Pricing (>200K tokens):**
- Input: $6.00 / MTok
- Output: $22.50 / MTok

**Cache Pricing:**
- Cache Write (5m): $3.75 / MTok
- Cache Write (1h): $6.00 / MTok
- Cache Read: $0.30 / MTok

**Batch API**: $1.50 / $7.50 per 1M (50% off)

Best for: Coding, agentic workflows, balanced performance/cost

**SWE-bench Verified**: 77.2%

---

### Claude Haiku 4.5
**Model String**: `claude-haiku-4-5-20251001` or `claude-haiku-4.5`
- **Context Window**: 200,000 tokens
- **Max Output**: 64,000 tokens
- **Pricing**: $1.00 / $5.00 per 1M tokens
- **Cache Write (5m)**: $1.25 / MTok
- **Cache Write (1h)**: $2.00 / MTok
- **Cache Read**: $0.10 / MTok
- **Batch API**: $0.50 / $2.50 per 1M (50% off)

Best for: High-volume, low-latency tasks, real-time chat, classification

Near-frontier performance at 5x lower cost than Sonnet

---

## Claude 4.x Family (Legacy - May 2025)

### Claude Opus 4.1 / Opus 4
**Model Strings**: `claude-opus-4-1-20250822`, `claude-opus-4-20250522`
- **Context Window**: 200,000 tokens
- **Max Output**: 64,000 tokens
- **Pricing**: $15.00 / $75.00 per 1M tokens (3x more expensive than 4.5!)
- **Batch API**: $7.50 / $37.50 per 1M

⚠️ **Recommendation**: Use Opus 4.5 instead - same or better quality at 66% lower cost

### Claude Sonnet 4
**Model String**: `claude-sonnet-4-20250514`
- **Context Window**: 200,000 tokens (1M beta)
- **Pricing**: $3.00 / $15.00 per 1M tokens
- Same pricing as Sonnet 4.5

---

## Older Models (Still Available)

### Claude Haiku 3.5
**Model String**: `claude-3-5-haiku-20241022`
- **Pricing**: $0.80 / $4.00 per 1M tokens

### Claude Haiku 3
**Model String**: `claude-3-haiku-20240307`
- **Pricing**: $0.25 / $1.25 per 1M tokens
- Budget champion for high-volume classification

---

## Key Features

### Extended Thinking
Available on: Opus 4.5, Sonnet 4.5, Haiku 4.5, Opus 4, Opus 4.1, Sonnet 4

```javascript
{
  "model": "claude-sonnet-4-5-20250929",
  "thinking": {
    "type": "enabled",
    "budget_tokens": 10000  // minimum 1024
  }
}
```

**Billing**: Thinking tokens billed as output tokens at standard rate

### Prompt Caching
Two cache durations available:
- **5-minute cache**: 1.25x base input price to write, 0.1x to read
- **1-hour cache**: 2x base input price to write, 0.1x to read

Up to **90% savings** on repeated context!

### Batch API
50% discount on all tokens. 24-hour processing window.

---

## Code Examples

### Standard Request
```javascript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic();

const response = await anthropic.messages.create({
  model: "claude-sonnet-4-5-20250929",
  max_tokens: 4096,
  messages: [
    { role: "user", content: "Hello, Claude!" }
  ]
});
```

### With Extended Thinking
```javascript
const response = await anthropic.messages.create({
  model: "claude-opus-4-5-20251101",
  max_tokens: 16000,
  thinking: {
    type: "enabled",
    budget_tokens: 10000
  },
  messages: [
    { role: "user", content: "Solve this complex problem..." }
  ]
});
```

### With Prompt Caching
```javascript
const response = await anthropic.messages.create({
  model: "claude-sonnet-4-5-20250929",
  max_tokens: 4096,
  system: [
    {
      type: "text",
      text: "Your very long system prompt here...",
      cache_control: { type: "ephemeral" }  // 5-minute cache
    }
  ],
  messages: [
    { role: "user", content: "Question about the cached context" }
  ]
});
```

### With Effort Parameter (Opus 4.5 only)
```javascript
const response = await anthropic.messages.create({
  model: "claude-opus-4-5-20251101",
  max_tokens: 8000,
  effort: "medium",  // low, medium, high
  messages: [
    { role: "user", content: "Complex task..." }
  ]
});
```

---

## Model Selection Guide

| Use Case | Recommended Model | Reason |
|----------|-------------------|--------|
| Complex reasoning | Opus 4.5 | Best intelligence, effort control |
| Coding (general) | Sonnet 4.5 | Best coding benchmarks, good cost |
| Agentic workflows | Sonnet 4.5 | Excellent tool use, fast |
| High-volume chat | Haiku 4.5 | Fast, cheap, near-frontier |
| Budget classification | Haiku 3 | Cheapest option |
| Long documents (1M) | Sonnet 4.5 (beta) | 1M context available |

---

## Platform Availability

All Claude models available on:
- **Anthropic API** (direct)
- **AWS Bedrock**
- **Google Vertex AI**
- **Microsoft Azure Foundry**

**Regional vs Global Endpoints** (4.5 models):
- Global: Standard pricing, dynamic routing
- Regional: +10% premium, guaranteed data locality
