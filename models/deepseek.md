# DeepSeek Models

## Current Models (DeepSeek-V3.2 - December 2025)

DeepSeek offers two API endpoints that both use the V3.2 model with different modes:

### deepseek-chat (Non-thinking Mode)
**Model String**: `deepseek-chat`
**Version**: DeepSeek-V3.2
- **Context Window**: 128,000 tokens
- **Max Output**: 8,000 tokens (default: 4K)
- **Pricing**:
  - Input (cache hit): $0.028 / 1M tokens
  - Input (cache miss): $0.28 / 1M tokens
  - Output: $0.42 / 1M tokens

Best for: General tasks, classification, summarization, extraction, agent pipelines

**Features Supported**:
- ✅ JSON Output
- ✅ Tool Calls
- ✅ Chat Prefix Completion (Beta)
- ✅ FIM Completion (Beta)

### deepseek-reasoner (Thinking Mode)
**Model String**: `deepseek-reasoner`
**Version**: DeepSeek-V3.2 with Chain-of-Thought
- **Context Window**: 128,000 tokens
- **Max CoT Tokens**: 32,000
- **Max Output**: 64,000 tokens (default: 32K)
- **Pricing**: Same as deepseek-chat
  - Input (cache hit): $0.028 / 1M tokens
  - Input (cache miss): $0.28 / 1M tokens
  - Output: $0.42 / 1M tokens (includes CoT tokens)

Best for: Math, logic, code analysis, complex reasoning

**Features Supported**:
- ✅ JSON Output
- ✅ Tool Calls
- ✅ Chat Prefix Completion (Beta)
- ❌ FIM Completion

---

## Previous Versions

### DeepSeek-R1 (January 2025)
**Model String**: `deepseek-r1` (may still be available)
- 671B parameters (37B activated per token)
- Mixture-of-Experts architecture
- Visible chain-of-thought reasoning
- MIT licensed (open source)

### DeepSeek-V3 (December 2024)
- 671B total parameters, 37B activated
- Multi-head Latent Attention (MLA)
- Trained on 14.8T tokens
- Training cost: only $5.58M

---

## Key Advantages

### Extremely Low Pricing
DeepSeek is **10-30x cheaper** than competitors:

| Model | Input | Output |
|-------|-------|--------|
| DeepSeek-chat | $0.28 | $0.42 |
| GPT-5 | $1.25 | $10.00 |
| Claude Sonnet 4.5 | $3.00 | $15.00 |

### Automatic Context Caching
- 90% discount on cache hits ($0.028 vs $0.28)
- Caching happens automatically
- No special configuration needed

### OpenAI-Compatible API
Drop-in replacement for OpenAI:
```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  baseURL: 'https://api.deepseek.com',
  apiKey: process.env.DEEPSEEK_API_KEY
});
```

### Open Source (MIT License)
- Run locally on your infrastructure
- No API costs for self-hosted
- Full control over data

---

## Code Examples

### Basic Chat
```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  baseURL: 'https://api.deepseek.com',
  apiKey: process.env.DEEPSEEK_API_KEY
});

const response = await client.chat.completions.create({
  model: 'deepseek-chat',
  messages: [
    { role: 'system', content: 'You are a helpful assistant.' },
    { role: 'user', content: 'Hello!' }
  ],
  max_tokens: 4096
});
```

### Reasoning Mode (R1)
```javascript
const response = await client.chat.completions.create({
  model: 'deepseek-reasoner',
  messages: [
    { role: 'user', content: 'Solve this math problem step by step: ...' }
  ],
  max_tokens: 8000
});

// Response includes visible chain-of-thought reasoning
```

### With Tool Calls
```javascript
const response = await client.chat.completions.create({
  model: 'deepseek-chat',
  messages: [
    { role: 'user', content: 'What is the weather in Tokyo?' }
  ],
  tools: [
    {
      type: 'function',
      function: {
        name: 'get_weather',
        description: 'Get current weather',
        parameters: {
          type: 'object',
          properties: {
            location: { type: 'string' }
          },
          required: ['location']
        }
      }
    }
  ]
});
```

### JSON Output
```javascript
const response = await client.chat.completions.create({
  model: 'deepseek-chat',
  messages: [
    { role: 'user', content: 'Extract entities from this text...' }
  ],
  response_format: { type: 'json_object' }
});
```

### Python Example
```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.deepseek.com",
    api_key=os.environ["DEEPSEEK_API_KEY"]
)

response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "user", "content": "Hello!"}
    ]
)
print(response.choices[0].message.content)
```

---

## Model Selection Guide

| Use Case | Model | Reason |
|----------|-------|--------|
| General chat | deepseek-chat | Fast, cheap |
| Math/logic | deepseek-reasoner | Chain-of-thought |
| Code tasks | deepseek-chat | Good at code |
| Budget operations | deepseek-chat | 10x cheaper than GPT |
| Self-hosting | DeepSeek-V3/R1 | MIT license |

---

## When to Use DeepSeek

**Best for:**
- High-volume, cost-sensitive applications
- Math and reasoning tasks (R1)
- Code generation and analysis
- Teams with infrastructure for self-hosting
- Projects requiring open-source models

**Consider alternatives for:**
- Cutting-edge performance (use GPT-5.2, Opus 4.5)
- Multimodal tasks (DeepSeek is text-only)
- Enterprise compliance requirements
- When you need maximum reasoning quality

---

## API Details

**Base URL**: `https://api.deepseek.com`

**Rate Limits**: Check dashboard for current limits

**Context Caching**: Automatic, no configuration needed

**Batch API**: 33% discount, 12-hour completion window

**Free Tier**: First 50-200M tokens free (varies by model)

---

## Self-Hosting Options

DeepSeek models are MIT licensed and available on Hugging Face:
- `deepseek-ai/DeepSeek-V3`
- `deepseek-ai/DeepSeek-R1`

Hardware requirements for V3/R1 (671B params):
- Minimum: 8x A100 80GB or equivalent
- Recommended: 8x H100 for good performance

For smaller deployments, consider distilled versions.
