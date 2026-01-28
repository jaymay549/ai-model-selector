# Reasoning Use Case Guide

## Model Recommendations

### Math & Quantitative Reasoning
**Top Picks**:
1. GPT-5.2 (xhigh effort) - 100% on AIME 2025
2. GPT-5.2 Pro - Maximum reasoning capability
3. o3 - Optimized for math/science

| Model | AIME 2025 | Best For |
|-------|-----------|----------|
| GPT-5.2 (xhigh) | 100% | Complex math |
| o3 | 90%+ | Fast math reasoning |
| DeepSeek-R1 | 79.8% | Budget math |
| Claude Opus 4.5 | High 80s% | Multi-step reasoning |

### Scientific Reasoning
**Top Pick**: GPT-5.2 or Claude Opus 4.5

| Model | GPQA Diamond | Best For |
|-------|--------------|----------|
| GPT-5.2 | ~92-93% | Hard science questions |
| Claude Opus 4.5 | ~85%+ | Research synthesis |
| Gemini 2.5 Pro | ~80%+ | Scientific analysis |

### Multi-Step Planning & Analysis
**Top Pick**: Claude Opus 4.5 with extended thinking

Opus 4.5 excels at sustained reasoning over many steps.

### Budget Reasoning
**Top Pick**: DeepSeek-reasoner

Visible chain-of-thought at $0.28/$0.42 per 1M tokens.

---

## Configuration for Reasoning

### GPT-5.2 Reasoning Effort
```javascript
// Maximum reasoning for hard problems
const response = await fetch("https://api.openai.com/v1/responses", {
  body: JSON.stringify({
    model: "gpt-5.2",
    reasoning: { effort: "xhigh" },  // Maximum thinking
    input: [{
      role: "user",
      content: "Prove that there are infinitely many primes..."
    }]
  })
});

// For simpler problems, use lower effort
{
  reasoning: { effort: "medium" }  // Saves tokens
}
```

### Claude Extended Thinking
```javascript
const response = await anthropic.messages.create({
  model: "claude-opus-4-5-20251101",
  max_tokens: 32000,
  thinking: {
    type: "enabled",
    budget_tokens: 20000  // More budget for hard problems
  },
  messages: [{
    role: "user",
    content: "Analyze this complex scenario..."
  }]
});
```

### DeepSeek Reasoning Mode
```javascript
const response = await client.chat.completions.create({
  model: "deepseek-reasoner",  // Thinking mode
  messages: [{
    role: "user",
    content: "Solve step by step: ..."
  }],
  max_tokens: 8000
});

// Response includes visible CoT reasoning
```

### Gemini Thinking Mode
```javascript
const model = genAI.getGenerativeModel({
  model: "gemini-2.5-pro",
  generationConfig: {
    thinkingConfig: {
      thinkingBudget: 15000
    }
  }
});
```

---

## Prompting Patterns for Reasoning

### Step-by-Step Reasoning
```
Solve this problem step by step. Show your work clearly.

Before giving your final answer:
1. Identify what type of problem this is
2. List the relevant information
3. Work through each step
4. Verify your answer

Problem: [your problem here]
```

### Self-Verification
```
Solve this problem, then verify your solution by:
1. Checking your arithmetic
2. Substituting your answer back
3. Considering edge cases
4. Looking for alternative approaches

Problem: [your problem here]
```

### Multi-Perspective Analysis
```
Analyze this from multiple perspectives:

1. First, consider the obvious interpretation
2. Then, consider alternative interpretations
3. Identify potential flaws in each approach
4. Synthesize into a final conclusion

Topic: [your topic here]
```

---

## Cost Comparison for Reasoning

| Model | Input | Output | Notes |
|-------|-------|--------|-------|
| GPT-5.2 | $1.75 | $14.00 | Best benchmark scores |
| GPT-5.2 Pro | $21.00 | $168.00 | Maximum capability |
| Claude Opus 4.5 | $5.00 | $25.00 | Best multi-step |
| Gemini 2.5 Pro | $1.25 | $10.00 | Good value |
| DeepSeek-reasoner | $0.28 | $0.42 | Budget champion |

**Note**: Extended thinking/reasoning tokens count toward output costs.

---

## Reasoning Task Routing

```javascript
function selectReasoningModel(task) {
  if (task.type === 'math' && task.difficulty === 'competition') {
    return { model: 'gpt-5.2', reasoning: { effort: 'xhigh' } };
  }
  
  if (task.type === 'math' && task.difficulty === 'standard') {
    return { model: 'gpt-5.2', reasoning: { effort: 'high' } };
  }
  
  if (task.type === 'multi-step-planning') {
    return { 
      model: 'claude-opus-4-5-20251101',
      thinking: { type: 'enabled', budget_tokens: 15000 }
    };
  }
  
  if (task.budget === 'low') {
    return { model: 'deepseek-reasoner' };
  }
  
  // Default: good balance
  return { model: 'gpt-5.2', reasoning: { effort: 'medium' } };
}
```

---

## When NOT to Use Heavy Reasoning

Save tokens by using lower reasoning effort for:
- Simple factual questions
- Basic classification
- Straightforward summaries
- Chat/conversation

```javascript
// Don't waste tokens on simple questions
{
  model: "gpt-5.2",
  reasoning: { effort: "none" }  // Fast mode
}
```

---

## Best Practices

1. **Start with medium effort**, increase if needed
2. **Set thinking budgets** appropriately - don't over-allocate
3. **Use visible reasoning** (DeepSeek) to debug model thinking
4. **Verify outputs** for math - even best models make errors
5. **Consider cost** - xhigh effort costs significantly more tokens
