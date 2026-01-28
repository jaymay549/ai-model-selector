# DeepSeek Models Reference

> Last verified: January 2026

## Model Overview

| Model | Context | Max Output | Max CoT | Vision | Thinking | Best For |
|-------|---------|------------|---------|--------|----------|----------|
| **DeepSeek V3.1** | 128K | 8K | - | ❌ | Optional | General, coding |
| **DeepSeek V3** | 64K | 8K | - | ❌ | ❌ | Budget general |
| **DeepSeek R1** | 64K | 8K | 64K | ❌ | ✅ | Complex reasoning |
| **DeepSeek R1-0528** | 128K | 32K | 32K | ❌ | ✅ | Latest reasoning |

## Model String Reference

```python
MODELS = {
    # DeepSeek Chat (V3 family)
    "deepseek-chat": "deepseek-chat",           # Points to latest V3
    "deepseek-v3": "deepseek-chat",             # Alias
    "deepseek-v3.1": "deepseek-v3.1",           # If available
    "deepseek-v3.2": "deepseek-v3.2",           # Latest with thinking toggle
    
    # DeepSeek Reasoner (R1 family)
    "deepseek-reasoner": "deepseek-reasoner",   # Points to latest R1
    "deepseek-r1": "deepseek-r1",               # Original
    "deepseek-r1-0528": "deepseek-r1-0528",     # May 2025 update
}
```

## API Implementation

### Setup (OpenAI-compatible)

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_DEEPSEEK_API_KEY",
    base_url="https://api.deepseek.com"
)
```

### Basic Chat (V3)

```python
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ],
    max_tokens=4096,
    temperature=0.7,
    stream=True
)

for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Reasoning Model (R1)

```python
response = client.chat.completions.create(
    model="deepseek-reasoner",
    messages=[
        {"role": "user", "content": "Solve this step by step: What is 23^4?"}
    ],
    max_tokens=8192,  # Include space for reasoning
    stream=True
)

# R1 includes reasoning in response
for chunk in response:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end="")
```

### V3.2 with Thinking Toggle

```python
# Enable thinking mode (similar to o1)
response = client.chat.completions.create(
    model="deepseek-v3.2",
    messages=[
        {"role": "user", "content": "Complex reasoning task..."}
    ],
    extra_body={
        "enable_thinking": True
    },
    stream=True
)

# Response includes reasoning_content field
```

### Accessing Reasoning Content

```python
# Non-streaming to get full reasoning
response = client.chat.completions.create(
    model="deepseek-reasoner",
    messages=[{"role": "user", "content": "Solve..."}],
    max_tokens=8192,
    stream=False
)

# Check usage for reasoning tokens
print(f"Reasoning tokens: {response.usage.completion_tokens}")

# Reasoning is embedded in the response content
print(response.choices[0].message.content)
```

## Pricing (per 1M tokens)

| Model | Input | Output | Cache Hit |
|-------|-------|--------|-----------|
| **DeepSeek V3/Chat** | $0.27 | $1.10 | $0.014 |
| **DeepSeek R1** | $0.55 | $2.19 | - |

**Extremely competitive pricing** - often 10-20x cheaper than competitors.

## Context Caching

DeepSeek automatically caches repeated prefixes:

```python
# First request - full price
response1 = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": long_system_prompt},  # Cached
        {"role": "user", "content": "Question 1"}
    ]
)

# Second request - cache hit on system prompt
response2 = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": long_system_prompt},  # Cache hit!
        {"role": "user", "content": "Question 2"}
    ]
)

# Check cache usage
print(f"Cache hits: {response2.usage.prompt_cache_hit_tokens}")
print(f"Cache misses: {response2.usage.prompt_cache_miss_tokens}")
```

Cache is in 64-token chunks, persists for a short time.

## Common Gotchas

1. **Context window is INPUT + OUTPUT combined**
   - 64K total for V3, not 64K input + 8K output
   - Plan accordingly for long conversations

2. **Output defaults to 4K, max 8K**
   - Must explicitly set `max_tokens` for longer outputs
   - R1-0528 supports up to 32K output

3. **Reasoning tokens count toward output**
   - R1's chain-of-thought is part of the response
   - Budget for longer outputs when using reasoning

4. **No vision support**
   - DeepSeek models are text-only
   - Use other providers for multimodal

5. **Rate limiting during high traffic**
   - No fixed limits, but throttling occurs
   - May get 429 errors during peak times

6. **Thinking mode is non-standard**
   - `enable_thinking` in `extra_body`
   - Not part of OpenAI-compatible spec

7. **Azure/AWS hosting available**
   - Higher rate limits, different pricing
   - Context may be longer (128K on Azure)

## Alternative Hosting

### Alibaba Model Studio
```python
client = OpenAI(
    api_key="YOUR_DASHSCOPE_KEY",
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

# V3.2 with thinking toggle
response = client.chat.completions.create(
    model="deepseek-v3.2",
    messages=[...],
    extra_body={"enable_thinking": True}
)
```

### AWS Bedrock
```python
import boto3

client = boto3.client("bedrock-runtime", region_name="us-west-2")

# Use cross-region inference profile
response = client.invoke_model(
    modelId="us.deepseek.r1-v1:0",
    body=json.dumps({
        "prompt": "<｜begin▁of▁sentence｜><｜User｜>Hello<｜Assistant｜>",
        "max_tokens": 8192,  # Max 8192 for quality, API accepts 32K
        "temperature": 0.5
    })
)
```

## Best Practices

1. **For reasoning tasks**: Use R1-0528 with high `max_tokens`
2. **For cost efficiency**: Use V3 for simple tasks
3. **For long context**: Consider Azure hosting (128K)
4. **For caching**: Keep system prompts consistent
5. **For streaming**: Always use for better UX with R1
