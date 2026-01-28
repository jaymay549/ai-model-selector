# OpenAI Models

## GPT-5.2 Family (December 2025 - Current Flagship)

### GPT-5.2
**Model String**: `gpt-5.2`
- **Context Window**: 400,000 tokens
- **Max Output**: 128,000 tokens
- **Knowledge Cutoff**: August 31, 2025
- **Pricing**: $1.75 / $14.00 per 1M tokens (input/output)
- **Cached Input**: $0.175 per 1M (90% discount)
- **Reasoning Effort**: `none` (default), `low`, `medium`, `high`, `xhigh`

Best for: Complex reasoning, coding, agentic tasks, long-context analysis

### GPT-5.2 Pro
**Model String**: `gpt-5.2-pro`
- **Context Window**: 400,000 tokens
- **Max Output**: 128,000 tokens
- **Pricing**: $21.00 / $168.00 per 1M tokens
- **Reasoning Effort**: `medium`, `high`, `xhigh` only
- **API**: Responses API only (not Chat Completions)
- **Note**: Requests may take several minutes; use background mode

Best for: Most difficult problems requiring maximum reasoning

### GPT-5.2-Codex
**Model String**: `gpt-5.2-codex`
- **Context Window**: 400,000 tokens
- **Max Output**: 128,000 tokens
- **Optimized for**: Agentic coding in Codex/Codex CLI
- **Features**: Context compaction, Windows support, enhanced cybersecurity

Best for: Long-horizon coding, large refactors, migrations

### GPT-5.2-Chat-Latest
**Model String**: `gpt-5.2-chat-latest`
- **Context Window**: 128,000 tokens (reduced)
- **Max Output**: 16,384 tokens
- **Same pricing as GPT-5.2**

Used in ChatGPT for "GPT-5.2 Instant" mode

---

## GPT-5.1 Family (November 2025)

### GPT-5.1
**Model String**: `gpt-5.1` or `gpt-5.1-2025-11-13`
- **Context Window**: 400,000 tokens
- **Max Output**: 128,000 tokens
- **Pricing**: $1.25 / $10.00 per 1M tokens
- **Reasoning Effort**: `none` (default), `low`, `medium`, `high`

### GPT-5.1-Codex / GPT-5.1-Codex-Max
**Model Strings**: `gpt-5.1-codex`, `gpt-5.1-codex-max`
- Optimized for agentic coding
- Max variant for long-running tasks

---

## GPT-5 Family (August 2025)

### GPT-5
**Model String**: `gpt-5` or `gpt-5-2025-08-07`
- **Context Window**: 400,000 tokens
- **Max Output**: 128,000 tokens
- **Pricing**: $1.25 / $10.00 per 1M tokens
- **Reasoning Effort**: `minimal`, `low`, `medium` (default), `high`

### GPT-5 Mini
**Model String**: `gpt-5-mini` or `gpt-5-mini-2025-08-07`
- **Context Window**: 400,000 tokens
- **Pricing**: Lower than GPT-5

### GPT-5 Nano
**Model String**: `gpt-5-nano` or `gpt-5-nano-2025-08-07`
- **Context Window**: 400,000 tokens
- Fastest, cheapest GPT-5 variant
- Good for summarization, classification

---

## GPT-4.1 Family (April 2025)

### GPT-4.1
**Model String**: `gpt-4.1` or `gpt-4.1-2025-04-14`
- **Context Window**: 1,000,000 tokens
- **Max Output**: 32,768 tokens
- **Pricing**: $2.00 / $8.00 per 1M tokens
- **Supports**: Temperature, top_p, all standard parameters

### GPT-4.1-mini
**Model String**: `gpt-4.1-mini` or `gpt-4.1-mini-2025-04-14`
- **Context Window**: 1,000,000 tokens
- **Pricing**: Lower cost option

### GPT-4.1-nano
**Model String**: `gpt-4.1-nano` or `gpt-4.1-nano-2025-04-14`
- **Context Window**: 1,000,000 tokens
- Fastest GPT-4.1 variant

---

## O-Series Reasoning Models

### o3
**Model String**: `o3`
- **Context Window**: 200,000 tokens
- **Max Output**: 100,000 tokens
- Optimized for math, science, coding, visual reasoning

### o3-pro
**Model String**: `o3-pro`
- More compute for harder problems
- Use background mode to avoid timeouts

### o4-mini
**Model String**: `o4-mini` or `o4-mini-2025-04-16`
- **Context Window**: 200,000 tokens
- Fast, cost-effective reasoning
- Succeeded by GPT-5 mini

---

## GPT-4o Family (Legacy but still available)

### GPT-4o
**Model String**: `gpt-4o`
- **Context Window**: 128,000 tokens
- **Max Output**: 16,384 tokens
- **Pricing**: $2.50 / $10.00 per 1M tokens
- Multimodal (text + images)

### GPT-4o-mini
**Model String**: `gpt-4o-mini`
- **Context Window**: 128,000 tokens
- **Pricing**: $0.15 / $0.60 per 1M tokens
- Best budget option for simple tasks

---

## Critical Implementation Notes

### Reasoning Models Parameter Restrictions
**GPT-5.x, o-series models do NOT support these parameters:**
- `temperature`
- `top_p`
- `presence_penalty`
- `frequency_penalty`
- `logprobs`
- `top_logprobs`
- `logit_bias`
- `max_tokens` (use `max_completion_tokens` instead)

**Use instead:**
```javascript
{
  "model": "gpt-5.2",
  "reasoning": { "effort": "medium" },  // none, low, medium, high, xhigh
  "text": { "verbosity": "medium" },     // low, medium, high
  "max_completion_tokens": 4096
}
```

### API Selection
- **GPT-5.2**: Use Responses API for CoT passing between turns
- **GPT-5.2 Pro**: Responses API only
- **GPT-4.1, GPT-4o**: Chat Completions API works fine

### Batch API (50% discount)
Available for all models. Use for non-time-sensitive workloads:
- Input: 50% off
- Output: 50% off

### Code Example (GPT-5.2)
```javascript
const response = await fetch("https://api.openai.com/v1/responses", {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${OPENAI_API_KEY}`,
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    model: "gpt-5.2",
    input: [
      { role: "user", content: "Solve this problem step by step..." }
    ],
    reasoning: { effort: "high" },
    max_output_tokens: 16000
  })
});
```

### Code Example (GPT-4.1 - Chat Completions)
```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4.1",
  messages: [
    { role: "system", content: "You are a helpful assistant." },
    { role: "user", content: "Hello!" }
  ],
  temperature: 0.7,
  max_tokens: 4096
});
```
