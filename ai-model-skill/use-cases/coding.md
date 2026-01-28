# Coding Use Case Guide

## Model Recommendations

### Agentic Coding (Long-running tasks, refactors, migrations)
**Top Pick**: Claude Sonnet 4.5 or GPT-5.2-Codex

| Model | SWE-bench | Best For |
|-------|-----------|----------|
| Claude Opus 4.5 | 80.9% | Most complex, multi-repo tasks |
| GPT-5.2-Codex | ~78% | Long-horizon, Windows support |
| Claude Sonnet 4.5 | 77.2% | Daily coding, best balance |
| GPT-5.1-Codex-Max | 77.9% | Alternative to 5.2 |

### Code Review & Analysis
**Top Pick**: Claude Opus 4.5

Opus excels at understanding architecture and catching subtle issues.

### Quick Edits & Simple Tasks
**Top Pick**: Claude Haiku 4.5 or GPT-4o-mini

Fast, cheap, good enough for simple changes.

### Budget Coding
**Top Pick**: DeepSeek-chat

At $0.28/$0.42 per 1M tokens, it's 10x cheaper than alternatives with decent coding ability.

---

## Tool Configurations

### Claude Code / Cursor
```json
{
  "model": "claude-sonnet-4-5-20250929",
  "max_tokens": 16000,
  "temperature": 0  // Claude 4.5 supports this
}
```

For complex tasks, use extended thinking:
```json
{
  "model": "claude-sonnet-4-5-20250929",
  "thinking": {
    "type": "enabled",
    "budget_tokens": 10000
  }
}
```

### OpenAI Codex CLI
```bash
# Use GPT-5.2-Codex for best results
codex --model gpt-5.2-codex

# For complex tasks
codex --model gpt-5.2 --reasoning-effort high
```

### Continue.dev / Other Tools
```json
{
  "models": [
    {
      "title": "Claude Sonnet 4.5",
      "provider": "anthropic",
      "model": "claude-sonnet-4-5-20250929"
    },
    {
      "title": "GPT-5.2",
      "provider": "openai", 
      "model": "gpt-5.2"
    }
  ]
}
```

---

## Best Practices

### 1. Use Extended Thinking for Complex Tasks
```javascript
// Claude with extended thinking
const response = await anthropic.messages.create({
  model: "claude-sonnet-4-5-20250929",
  max_tokens: 16000,
  thinking: {
    type: "enabled",
    budget_tokens: 8000
  },
  messages: [{
    role: "user",
    content: "Refactor this module to use dependency injection..."
  }]
});
```

### 2. GPT-5.2 Reasoning Effort
```javascript
// Higher effort for harder problems
const response = await fetch("https://api.openai.com/v1/responses", {
  body: JSON.stringify({
    model: "gpt-5.2-codex",
    reasoning: { effort: "high" },  // none, low, medium, high, xhigh
    input: [{ role: "user", content: "Complex coding task..." }]
  })
});
```

### 3. Opus 4.5 Effort Parameter
```javascript
// Control compute for Opus
const response = await anthropic.messages.create({
  model: "claude-opus-4-5-20251101",
  effort: "medium",  // low, medium, high
  max_tokens: 8000,
  messages: [{ role: "user", content: "Architect this system..." }]
});
```

---

## Cost Comparison (per 1M tokens)

| Model | Input | Output | Notes |
|-------|-------|--------|-------|
| Claude Sonnet 4.5 | $3.00 | $15.00 | Best value for quality |
| Claude Opus 4.5 | $5.00 | $25.00 | When you need the best |
| GPT-5.2-Codex | $1.75 | $14.00 | Good for long sessions |
| Claude Haiku 4.5 | $1.00 | $5.00 | Quick edits |
| DeepSeek-chat | $0.28 | $0.42 | Budget option |

---

## Coding Task Routing

```javascript
function selectCodingModel(task) {
  if (task.complexity === 'simple' || task.type === 'quick-edit') {
    return 'claude-haiku-4-5-20251001';
  }
  
  if (task.complexity === 'high' || task.multiRepo || task.architecture) {
    return 'claude-opus-4-5-20251101';
  }
  
  if (task.longRunning || task.windows) {
    return 'gpt-5.2-codex';
  }
  
  // Default: best balance
  return 'claude-sonnet-4-5-20250929';
}
```

---

## IDE Integration Examples

### VS Code with Claude
```json
// settings.json
{
  "claude.model": "claude-sonnet-4-5-20250929",
  "claude.maxTokens": 16000
}
```

### Cursor Configuration
```json
{
  "cursor.aiModel": "claude-sonnet-4-5-20250929",
  "cursor.enableExtendedThinking": true
}
```

### Codeium/Copilot Alternatives
For budget setups, consider DeepSeek as a self-hosted option:
- MIT license, free to run
- Good coding performance
- Requires significant GPU resources
