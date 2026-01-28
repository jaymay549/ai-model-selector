# OpenAI Models Reference

> Last verified: January 2026

## Model Overview

| Model | Context | Max Output | Temperature | Vision | Tool Use | Best For |
|-------|---------|------------|-------------|--------|----------|----------|
| **GPT-4.1** | 1M | 32K | ✅ 0-2 | ✅ | ✅ | General, long context |
| **GPT-4.1-mini** | 1M | 32K | ✅ 0-2 | ✅ | ✅ | Fast general use |
| **GPT-4.1-nano** | 1M | 32K | ✅ 0-2 | ✅ | ✅ | High throughput |
| **GPT-4o** | 128K | 16K | ✅ 0-2 | ✅ | ✅ | Multimodal |
| **GPT-4o-mini** | 128K | 16K | ✅ 0-2 | ✅ | ✅ | Budget multimodal |
| **o3** | 200K | 100K | ❌ | ✅ | ✅ | Complex reasoning |
| **o4-mini** | 128K | 100K | ❌ | ✅ | ✅ | Fast reasoning |
| **o3-mini** | 200K | 100K | ❌ | ❌ | ✅ | Budget reasoning |
| **o1** | 200K | 100K | ✅ | ✅ | ✅ | Previous reasoning |
| **GPT-5** | 196K | 100K | ❌ | ✅ | ✅ | Frontier reasoning |
| **GPT-5-mini** | - | - | ❌ | ✅ | ✅ | Fast frontier |

## Model String Reference

```python
# Current recommended model strings
MODELS = {
    # GPT-4.1 family (1M context)
    "gpt-4.1": "gpt-4.1-2025-04-14",
    "gpt-4.1-mini": "gpt-4.1-mini-2025-04-14",
    "gpt-4.1-nano": "gpt-4.1-nano-2025-04-14",
    
    # GPT-4o family (128K context)
    "gpt-4o": "gpt-4o-2024-11-20",
    "gpt-4o-mini": "gpt-4o-mini-2024-07-18",
    
    # Reasoning models
    "o3": "o3",
    "o4-mini": "o4-mini",
    "o3-mini": "o3-mini-2025-01-31",
    "o1": "o1",
    "o3-pro": "o3-pro",
    
    # GPT-5 family
    "gpt-5": "gpt-5",
    "gpt-5-mini": "gpt-5-mini",
}
```

## ⚠️ Critical: Reasoning Model Constraints

Reasoning models (o-series, GPT-5) **DO NOT SUPPORT** these parameters:

```python
# ❌ WILL CAUSE ERRORS with o3, o4-mini, GPT-5
UNSUPPORTED_PARAMS = [
    "temperature",      # Fixed at 1
    "top_p",           # Fixed at 1
    "presence_penalty", # Fixed at 0
    "frequency_penalty",# Fixed at 0
    "logprobs",
    "top_logprobs",
    "logit_bias",
    "max_tokens",      # Use max_completion_tokens instead
]
```

**Use `reasoning_effort` instead of temperature:**
```python
# ✅ CORRECT for reasoning models
response = client.responses.create(
    model="gpt-5",
    reasoning={"effort": "medium"},  # low, medium, high
    input=[{"role": "user", "content": prompt}]
)
```

## Standard Model Implementation

### Chat Completions (GPT-4.1, GPT-4o)

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4.1",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ],
    max_tokens=4096,          # Max output tokens
    temperature=0.7,          # 0-2, lower = more deterministic
    top_p=1,                  # Nucleus sampling
    frequency_penalty=0,      # -2 to 2
    presence_penalty=0,       # -2 to 2
    stream=True               # Enable streaming
)

for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Reasoning Models (o-series, GPT-5)

```python
# ⚠️ Use Responses API for best results
response = client.responses.create(
    model="o3",
    reasoning={"effort": "high"},  # low, medium, high
    input=[
        {"role": "user", "content": "Solve this step by step: ..."}
    ]
)

print(response.output_text)

# Access reasoning tokens (billed as output)
print(f"Reasoning tokens: {response.usage.reasoning_tokens}")
```

### Vision

```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "What's in this image?"},
            {
                "type": "image_url",
                "image_url": {
                    "url": f"data:image/jpeg;base64,{base64_image}",
                    "detail": "high"  # low, high, auto
                }
            }
        ]
    }],
    max_tokens=1000
)
```

### Tool Use / Function Calling

```python
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get current weather",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string"},
                "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
            },
            "required": ["location"]
        }
    }
}]

response = client.chat.completions.create(
    model="gpt-4.1",
    messages=[{"role": "user", "content": "Weather in Paris?"}],
    tools=tools,
    tool_choice="auto"  # auto, none, or specific function
)

# Handle tool calls
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    # Execute function and return result
```

## Pricing (per 1M tokens)

| Model | Input | Output | Notes |
|-------|-------|--------|-------|
| GPT-4.1 | $2.00 | $8.00 | Best value for 1M context |
| GPT-4.1-mini | $0.40 | $1.60 | |
| GPT-4.1-nano | $0.10 | $0.40 | Cheapest |
| GPT-4o | $2.50 | $10.00 | |
| GPT-4o-mini | $0.15 | $0.60 | |
| o3 | $10.00 | $40.00 | + reasoning tokens |
| o4-mini | $1.10 | $4.40 | + reasoning tokens |
| o3-mini | $1.10 | $4.40 | + reasoning tokens |

## Common Gotchas

1. **`max_tokens` vs `max_completion_tokens`**
   - Standard models: `max_tokens`
   - Reasoning models: `max_completion_tokens`

2. **System messages with o-series**
   - o3, o4-mini, o1 treat system messages as developer messages
   - Don't use both system AND developer messages in same request

3. **Context window includes reasoning tokens**
   - Reasoning tokens count against context but aren't visible
   - Budget accordingly for long conversations

4. **Streaming with o3**
   - Limited access only
   - o4-mini and o3-mini support streaming

5. **GPT-5 reasoning modes**
   - Default is `none` (fast)
   - Use `minimal`, `low`, `medium`, `high` for reasoning
