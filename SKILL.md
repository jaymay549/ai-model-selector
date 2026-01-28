---
name: ai-model-selector
description: "Use when selecting AI models, configuring API parameters, or implementing LLM calls. Covers OpenAI, Anthropic, Google, DeepSeek, and embedding models with specs, gotchas, and code templates."
version: 1.0.0
license: MIT
author: Jason
compatibility:
  - claude-code
  - cursor
  - codex
  - opencode
tags:
  - llm
  - api
  - openai
  - anthropic
  - gemini
  - deepseek
  - embeddings
  - rag
---

# AI Model Selection & Implementation Guide

## Quick Decision Tree

```
What's your primary task?
│
├── Complex reasoning / Math / Science / Multi-step planning
│   ├── Budget priority → DeepSeek R1 ($0.55/1M input)
│   ├── Best quality → OpenAI o3 or GPT-5
│   └── Balanced → Claude Opus 4.5
│
├── Coding / Software engineering
│   ├── Agentic coding → Claude Sonnet 4.5 or GPT-5.1-Codex
│   ├── Fast iteration → Claude Haiku 4.5 or GPT-4.1-mini
│   └── Budget → DeepSeek V3
│
├── General chat / Assistants
│   ├── High quality → Claude Sonnet 4.5 or GPT-4.1
│   ├── Fast/cheap → Claude Haiku 4.5 or GPT-4.1-nano
│   └── Free tier → Gemini 2.5 Flash
│
├── Vision / Image understanding
│   ├── Best quality → Claude Opus 4.5 or GPT-4o
│   └── Fast → Gemini 2.5 Flash
│
├── Long context (>200K tokens)
│   ├── Up to 1M tokens → GPT-4.1, Gemini 2.5 Pro
│   └── Up to 200K → Claude models
│
└── Embeddings / RAG
    ├── General purpose → Voyage 3-large or text-embedding-3-large
    ├── Code → Voyage Code 3
    └── Budget → text-embedding-3-small
```

## Model Categories at a Glance

| Provider | Flagship | Balanced | Fast/Cheap | Reasoning |
|----------|----------|----------|------------|-----------|
| **Anthropic** | Opus 4.5 | Sonnet 4.5 | Haiku 4.5 | Extended thinking |
| **OpenAI** | GPT-5 | GPT-4.1 | GPT-4.1-nano | o3, o4-mini |
| **Google** | Gemini 2.5 Pro | Gemini 2.5 Flash | Flash-Lite | Thinking mode |
| **DeepSeek** | V3.1 | V3 | - | R1 |

## Critical Implementation Notes

### Reasoning Models (o-series, GPT-5) Don't Support:
- `temperature` (fixed at 1)
- `top_p`
- `presence_penalty` / `frequency_penalty`
- `logprobs`
- `max_tokens` (use `max_completion_tokens` instead)

### Context Window ≠ Max Output
Every model has separate limits. A 200K context window might only allow 8K output tokens.

### Pricing Varies by Context Length
Claude Sonnet 4/4.5 charges premium rates (2x) for requests >200K input tokens.

## File Structure

```
ai-model-skill/
├── SKILL.md                    # This file - overview and decision tree
├── models/
│   ├── openai.md              # GPT-4.1, GPT-5, o-series specs
│   ├── anthropic.md           # Claude 4.5 family specs
│   ├── google.md              # Gemini specs
│   ├── deepseek.md            # DeepSeek V3, R1 specs
│   └── embeddings.md          # Embedding model comparison
└── use-cases/
    ├── coding.md              # Best models for coding tasks
    ├── reasoning.md           # Complex reasoning tasks
    └── rag.md                 # Retrieval-augmented generation
```

## Quick Reference: When to Read Which File

| If you need... | Read |
|----------------|------|
| OpenAI model specs and params | `models/openai.md` |
| Claude API implementation | `models/anthropic.md` |
| Gemini setup | `models/google.md` |
| DeepSeek configuration | `models/deepseek.md` |
| Embedding model selection | `models/embeddings.md` |
| Coding assistant setup | `use-cases/coding.md` |
| Math/reasoning workloads | `use-cases/reasoning.md` |
| RAG pipeline models | `use-cases/rag.md` |

## Version History

- **1.0.0** (2026-01-28): Initial release with current model specs
