# Anthropic Claude Models Reference

> Last verified: January 2026

## Model Overview

| Model | Context | Max Output | Vision | Extended Thinking | Best For |
|-------|---------|------------|--------|-------------------|----------|
| **Claude Opus 4.5** | 200K | 64K | ✅ | ✅ | Frontier, complex reasoning |
| **Claude Sonnet 4.5** | 200K (1M beta) | 64K | ✅ | ✅ | Coding, agents, balanced |
| **Claude Haiku 4.5** | 200K | 64K | ✅ | ✅ | Fast, high-volume |
| **Claude Sonnet 4** | 200K (1M beta) | 64K | ✅ | ✅ | Previous generation |
| **Claude Opus 4.1** | 200K | 64K | ✅ | ✅ | Previous flagship |
| **Claude Opus 4** | 200K | 64K | ✅ | ✅ | Legacy |

## Model String Reference

```python
MODELS = {
    # Claude 4.5 family (current)
    "opus-4.5": "claude-opus-4-5-20251101",
    "sonnet-4.5": "claude-sonnet-4-5-20250929",
    "haiku-4.5": "claude-haiku-4-5-20251001",
    
    # Claude 4 family
    "sonnet-4": "claude-sonnet-4-20250514",
    "opus-4": "claude-opus-4-20250514",
    "opus-4.1": "claude-opus-4-1-20250819",
    
    # Legacy 3.x (still available)
    "sonnet-3.7": "claude-3-7-sonnet-20250219",
    "sonnet-3.5": "claude-3-5-sonnet-20241022",
    "haiku-3.5": "claude-3-5-haiku-20241022",
}
```

## API Implementation

### Basic Chat

```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    messages=[
        {"role": "user", "content": "Hello, Claude!"}
    ]
)

print(message.content[0].text)
```

### With System Prompt

```python
message = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    system="You are a helpful coding assistant.",
    messages=[
        {"role": "user", "content": "Write a Python function"}
    ]
)
```

### Streaming

```python
with client.messages.stream(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    messages=[{"role": "user", "content": "Tell me a story"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### Vision

```python
import base64

with open("image.jpg", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

message = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/jpeg",
                    "data": image_data
                }
            },
            {
                "type": "text",
                "text": "What's in this image?"
            }
        ]
    }]
)
```

### Tool Use

```python
tools = [{
    "name": "get_weather",
    "description": "Get the current weather in a location",
    "input_schema": {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "City and state, e.g. San Francisco, CA"
            }
        },
        "required": ["location"]
    }
}]

message = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in Paris?"}]
)

# Check for tool use
if message.stop_reason == "tool_use":
    tool_use = next(b for b in message.content if b.type == "tool_use")
    tool_name = tool_use.name
    tool_input = tool_use.input
    # Execute tool and continue conversation
```

### Extended Thinking

```python
# Enable extended thinking for complex reasoning
message = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # Max tokens for thinking
    },
    messages=[{"role": "user", "content": "Solve this complex problem..."}]
)

# Access thinking content
for block in message.content:
    if block.type == "thinking":
        print(f"Thinking: {block.thinking}")
    elif block.type == "text":
        print(f"Response: {block.text}")
```

### Effort Parameter (Opus 4.5 only)

```python
# Control computational effort
message = client.messages.create(
    model="claude-opus-4-5-20251101",
    max_tokens=4096,
    extra_body={
        "effort": "medium"  # low, medium, high
    },
    messages=[{"role": "user", "content": "Analyze this data..."}]
)
```

## Pricing (per 1M tokens)

| Model | Input | Output | Cache Write | Cache Read |
|-------|-------|--------|-------------|------------|
| **Opus 4.5** | $5.00 | $25.00 | $6.25 | $0.50 |
| **Sonnet 4.5** | $3.00 | $15.00 | $3.75 | $0.30 |
| **Haiku 4.5** | $1.00 | $5.00 | $1.25 | $0.10 |
| **Sonnet 4.5 (>200K)** | $6.00 | $22.50 | - | - |

## Special Features

### 1M Token Context (Beta)
```python
# Available for Sonnet 4 and Sonnet 4.5
# Requires beta header
message = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=64000,
    extra_headers={
        "anthropic-beta": "max-tokens-3-5-sonnet-2024-07-15"
    },
    messages=[{"role": "user", "content": very_long_document}]
)
```

### Prompt Caching
```python
# Cache system prompts for repeated use
message = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    system=[{
        "type": "text",
        "text": "You are a legal expert...",
        "cache_control": {"type": "ephemeral"}  # 5-min cache
    }],
    messages=[{"role": "user", "content": "Review this contract"}]
)
```

### Context Awareness (4.5 models)
Claude 4.5 models automatically track remaining context window capacity.

### Batch API (50% discount)
```python
# Submit batch requests for async processing
batch = client.batches.create(
    requests=[
        {"custom_id": "req1", "params": {...}},
        {"custom_id": "req2", "params": {...}}
    ]
)
```

## Common Gotchas

1. **Max tokens is OUTPUT only**
   - Unlike OpenAI, `max_tokens` only limits output
   - Context window handles input

2. **No temperature for extended thinking**
   - Temperature fixed when thinking is enabled

3. **Long context pricing**
   - Sonnet charges 2x for requests >200K input tokens

4. **Tool use response format**
   - Stop reason is `tool_use`, not `function_call`
   - Tool calls are in `content` array, not separate field

5. **Image token costs**
   - Images consume tokens based on size
   - ~1,600 tokens for a 1024x1024 image

6. **Haiku 4.5 vs Haiku 3.5**
   - 4.5 is 25% more expensive but significantly more capable
   - Consider for quality-sensitive high-volume workloads
