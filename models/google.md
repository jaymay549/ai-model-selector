# Google Gemini Models Reference

> Last verified: January 2026

## Model Overview

| Model | Context | Max Output | Vision | Thinking | Best For |
|-------|---------|------------|--------|----------|----------|
| **Gemini 2.5 Pro** | 1M (2M soon) | 64K | ✅ | ✅ | Complex reasoning, long context |
| **Gemini 2.5 Flash** | 1M | 8K | ✅ | ✅ | Fast, high-volume |
| **Gemini 2.5 Flash-Lite** | 1M | 8K | ✅ | ❌ | Budget, simple tasks |
| **Gemini 2.0 Flash** | 1M | 8K | ✅ | ❌ | Legacy, being deprecated |
| **Gemini 3 Pro** | 1M | 64K | ✅ | ✅ | Frontier (if available) |

## Model String Reference

```python
MODELS = {
    # Gemini 2.5 (current recommended)
    "gemini-2.5-pro": "gemini-2.5-pro",
    "gemini-2.5-flash": "gemini-2.5-flash",
    "gemini-2.5-flash-lite": "gemini-2.5-flash-lite",
    
    # Gemini 2.0 (being deprecated March 2026)
    "gemini-2.0-flash": "gemini-2.0-flash",
    "gemini-2.0-flash-lite": "gemini-2.0-flash-lite",
    
    # Stable snapshots
    "gemini-2.5-pro-stable": "gemini-2.5-pro-001",
    "gemini-2.5-flash-stable": "gemini-2.5-flash-001",
}
```

## API Implementation (Google AI Studio)

### Setup

```python
import google.generativeai as genai

genai.configure(api_key="YOUR_API_KEY")

model = genai.GenerativeModel("gemini-2.5-flash")
```

### Basic Chat

```python
response = model.generate_content("Explain quantum computing")
print(response.text)
```

### With Configuration

```python
generation_config = {
    "temperature": 0.7,
    "top_p": 0.95,
    "top_k": 40,
    "max_output_tokens": 8192,
}

model = genai.GenerativeModel(
    model_name="gemini-2.5-flash",
    generation_config=generation_config
)

response = model.generate_content("Write a poem")
```

### Multi-turn Chat

```python
chat = model.start_chat(history=[])

response = chat.send_message("Hello!")
print(response.text)

response = chat.send_message("What did I just say?")
print(response.text)
```

### Vision

```python
import PIL.Image

image = PIL.Image.open("image.jpg")

response = model.generate_content([
    "What's in this image?",
    image
])
print(response.text)
```

### Thinking Mode (2.5 models)

```python
# Enable thinking for complex reasoning
model = genai.GenerativeModel(
    model_name="gemini-2.5-flash",
    generation_config={
        "thinking_config": {
            "thinking_budget": 10000  # Max thinking tokens
        }
    }
)

response = model.generate_content("Solve this complex math problem...")

# Access thinking process if available
for part in response.candidates[0].content.parts:
    if hasattr(part, 'thought'):
        print(f"Thinking: {part.thought}")
    else:
        print(f"Response: {part.text}")
```

### Streaming

```python
response = model.generate_content(
    "Tell me a long story",
    stream=True
)

for chunk in response:
    print(chunk.text, end="", flush=True)
```

## Vertex AI Implementation

```python
import vertexai
from vertexai.generative_models import GenerativeModel

vertexai.init(project="your-project", location="us-central1")

model = GenerativeModel("gemini-2.5-pro")

response = model.generate_content(
    "Your prompt here",
    generation_config={
        "max_output_tokens": 8192,
        "temperature": 0.7,
    }
)
```

## REST API (Direct)

```bash
curl "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{"text": "Hello!"}]
    }],
    "generationConfig": {
      "temperature": 0.7,
      "maxOutputTokens": 8192
    }
  }'
```

## Pricing (per 1M tokens)

| Model | Input (<128K) | Input (>128K) | Output |
|-------|---------------|---------------|--------|
| **Gemini 2.5 Pro** | $1.25 | $2.50 | $5.00 |
| **Gemini 2.5 Flash** | $0.075 | $0.15 | $0.30 |
| **Gemini 2.5 Flash-Lite** | $0.0375 | $0.075 | $0.15 |

**Free tier available** in AI Studio with generous limits.

## Special Features

### Grounding with Google Search
```python
from vertexai.generative_models import Tool

search_tool = Tool.from_google_search_retrieval(
    google_search_retrieval={"dynamic_retrieval_config": {"mode": "MODE_DYNAMIC"}}
)

model = GenerativeModel(
    "gemini-2.5-flash",
    tools=[search_tool]
)
```

### Context Caching
```python
# Cache large documents for repeated queries
cached_content = genai.caching.create(
    model="gemini-2.5-flash",
    contents=[large_document],
    ttl=datetime.timedelta(hours=1)
)

model = genai.GenerativeModel.from_cached_content(cached_content)
```

### Code Execution
```python
model = genai.GenerativeModel(
    "gemini-2.5-flash",
    tools="code_execution"
)

response = model.generate_content("Calculate the 50th Fibonacci number")
```

### Multimodal Input
```python
# Audio, video, PDF support
response = model.generate_content([
    "Summarize this video",
    genai.upload_file("video.mp4")
])
```

## Common Gotchas

1. **Model naming conventions changed Sept 2025**
   - Stable models: `gemini-2.5-flash` (no suffix)
   - Specific versions: `gemini-2.5-flash-001`
   - Preview: `gemini-2.5-flash-preview`

2. **Thinking tokens aren't always visible**
   - Depends on model and configuration
   - Still billed even if not shown

3. **Long context pricing tiers**
   - Different rates above/below 128K tokens
   - Plan accordingly for large documents

4. **Deprecation timeline**
   - Gemini 2.0 Flash/Lite: March 31, 2026
   - Migrate to 2.5 versions

5. **Rate limits vary by tier**
   - Free tier: ~15 RPM, ~1M TPM
   - Paid tier: significantly higher

6. **Output token limits**
   - Flash models: 8K default, can request more
   - Pro models: up to 64K

7. **Global endpoint vs regional**
   - Some models only available in specific regions
   - Global endpoint routes automatically
