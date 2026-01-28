# Google Gemini Models

## Gemini 3 Family (Late 2025 - Preview)

### Gemini 3 Pro (Preview)
**Model String**: `gemini-3-pro-preview`
- **Context Window**: 1,000,000 tokens (2M coming)
- **Max Output**: 64,000 tokens
- **Pricing (≤200K context)**: $2.00 / $12.00 per 1M tokens
- **Pricing (>200K context)**: $4.00 / $18.00 per 1M tokens
- **Status**: Preview - pricing may settle around $1.50/$10 in stable release

Best for: Most complex reasoning, multimodal analysis, large document processing

**Note**: 80%+ better reasoning than Gemini 2.5 on complex tasks

---

## Gemini 2.5 Family (Current Stable)

### Gemini 2.5 Pro
**Model String**: `gemini-2.5-pro` or `gemini-2.5-pro-latest`
- **Context Window**: 1,000,000 tokens
- **Max Output**: 64,000 tokens
- **Pricing (≤200K context)**: $1.25 / $10.00 per 1M tokens
- **Pricing (>200K context)**: $2.50 / $15.00 per 1M tokens
- **Thinking Mode Output**: $3.50 / MTok (when enabled)

Best for: Complex reasoning, code, math, STEM, long document analysis

### Gemini 2.5 Flash
**Model String**: `gemini-2.5-flash` or `gemini-2.5-flash-latest`
- **Context Window**: 1,000,000 tokens
- **Max Output**: 65,536 tokens
- **Pricing**: $0.30 / $2.50 per 1M tokens (non-thinking)
- **Thinking Mode**: $0.15 input, $3.50 output per 1M
- **Cache Read**: $0.075 / MTok

Best for: Large-scale processing, low-latency tasks, balanced price/performance

### Gemini 2.5 Flash-Lite
**Model String**: `gemini-2.5-flash-lite`
- **Context Window**: 1,000,000 tokens
- **Pricing**: $0.10 / $0.40 per 1M tokens

Most cost-efficient model for high-throughput tasks

---

## Gemini 2.0 Family (Being Deprecated)

### Gemini 2.0 Flash
**Model String**: `gemini-2.0-flash`
- **Context Window**: 1,000,000 tokens
- **Status**: Deprecated - shutdown March 31, 2026
- Native tool use, multimodal

### Gemini 2.0 Flash-Lite
**Model String**: `gemini-2.0-flash-lite`
- **Status**: Deprecated - shutdown March 31, 2026

⚠️ **Migrate to Gemini 2.5 Flash or Flash-Lite**

---

## Specialized Models

### Gemini 2.5 Flash TTS
**Model String**: `gemini-2.5-flash-tts`
- Text-to-speech model
- Low-latency, controllable speech generation
- Preview status

### Imagen 4.0
**Model Strings**: `imagen-4.0-generate-001`, `imagen-4.0-ultra-generate-001`, `imagen-4.0-fast-generate-001`
- Latest image generation
- Better text rendering and quality

### Veo 3.1
**Model Strings**: `veo-3.1-generate-preview`, `veo-3.1-fast-generate-preview`
- Video generation (paid tier only)

---

## Free Tier

Google AI Studio offers generous free access:
- **Models**: Gemini 1.5 Pro, 2.5 Flash, Flash-Lite
- **Rate Limits**: 5-15 RPM depending on model
- **Token Limits**: 250,000 TPM, 1,000 requests/day

Perfect for prototyping and testing!

---

## Key Features

### Thinking Mode
Available on Gemini 2.5 Pro and Flash:
```javascript
{
  "model": "gemini-2.5-flash",
  "generationConfig": {
    "thinkingConfig": {
      "thinkingBudget": 10000  // tokens for reasoning
    }
  }
}
```

### Grounding with Google Search
Add real-time search data to responses:
- First 1,500 queries/day free on paid tiers
- Then $35 per 1,000 grounding queries

### Context Caching
Reduce costs for repeated prompts:
- Cache read at ~$0.075/MTok for Flash
- Significant savings for document-heavy workloads

---

## Code Examples

### Basic Request (Gemini Developer API)
```javascript
import { GoogleGenerativeAI } from "@google/generative-ai";

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
const model = genAI.getGenerativeModel({ model: "gemini-2.5-flash" });

const result = await model.generateContent("Hello, Gemini!");
console.log(result.response.text());
```

### With Thinking Mode
```javascript
const model = genAI.getGenerativeModel({
  model: "gemini-2.5-pro",
  generationConfig: {
    thinkingConfig: {
      thinkingBudget: 8000
    }
  }
});

const result = await model.generateContent("Complex reasoning task...");
```

### Multimodal (Image + Text)
```javascript
const model = genAI.getGenerativeModel({ model: "gemini-2.5-flash" });

const imagePart = {
  inlineData: {
    data: base64ImageData,
    mimeType: "image/jpeg"
  }
};

const result = await model.generateContent([
  "What's in this image?",
  imagePart
]);
```

### Vertex AI (Google Cloud)
```python
import vertexai
from vertexai.generative_models import GenerativeModel

vertexai.init(project="your-project", location="us-central1")
model = GenerativeModel("gemini-2.5-pro")

response = model.generate_content("Hello!")
print(response.text)
```

---

## Model Selection Guide

| Use Case | Recommended Model | Reason |
|----------|-------------------|--------|
| Complex reasoning | Gemini 2.5 Pro | Best thinking capabilities |
| High-volume processing | Gemini 2.5 Flash | Fast, cheap, 1M context |
| Budget tasks | Gemini 2.5 Flash-Lite | Cheapest ($0.10/$0.40) |
| Free prototyping | Gemini 1.5 Pro (free) | No cost, good for dev |
| Cutting-edge | Gemini 3 Pro (preview) | Latest capabilities |
| Image generation | Imagen 4.0 | Best quality |
| Video generation | Veo 3.1 | Latest video model |

---

## Pricing Comparison (per 1M tokens)

| Model | Input | Output | Notes |
|-------|-------|--------|-------|
| Gemini 3 Pro | $2.00 | $12.00 | Preview, ≤200K |
| Gemini 2.5 Pro | $1.25 | $10.00 | ≤200K context |
| Gemini 2.5 Flash | $0.30 | $2.50 | Non-thinking |
| Gemini 2.5 Flash-Lite | $0.10 | $0.40 | Cheapest |
| Flash Thinking | $0.15 | $3.50 | With thinking enabled |

---

## Platform Availability

- **Google AI Studio** (ai.google.dev) - Free tier available
- **Vertex AI** (Google Cloud) - Enterprise features
- **Firebase** - Mobile/web integration

All platforms support same models with slightly different pricing on Vertex AI.
