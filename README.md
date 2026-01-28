# AI Model Selector Skill

A comprehensive guide to AI model selection and API implementation for AI coding agents. **Updated January 2026.**

## What's Included

| File | Description |
|------|-------------|
| `SKILL.md` | Main entry point with decision tree |
| `models/openai.md` | GPT-5.2, GPT-5.1, GPT-4.1, o-series specs |
| `models/anthropic.md` | Claude Opus/Sonnet/Haiku 4.5 |
| `models/google.md` | Gemini 2.5/3 Pro/Flash |
| `models/deepseek.md` | DeepSeek V3.2, R1 (cheapest option) |
| `models/embeddings.md` | Voyage, OpenAI, Gemini embeddings |
| `use-cases/coding.md` | Best models for coding tasks |
| `use-cases/reasoning.md` | Math, science, planning |
| `use-cases/rag.md` | RAG pipeline guidance |

## Key Features

- **Decision tree** for quick model selection
- **Exact model strings** and API parameters
- **Critical gotchas** (e.g., GPT-5.x doesn't support temperature)
- **Code snippets** for each provider
- **Current pricing** (January 2026)
- **Use-case specific** configurations

## Quick Reference (January 2026)

### Flagship Models
| Model | Context | Price (in/out per 1M) |
|-------|---------|----------------------|
| GPT-5.2 | 400K | $1.75 / $14 |
| Claude Opus 4.5 | 200K | $5 / $25 |
| Claude Sonnet 4.5 | 200K (1M beta) | $3 / $15 |
| Gemini 2.5 Pro | 1M | $1.25 / $10 |

### Budget Options
| Model | Context | Price (in/out per 1M) |
|-------|---------|----------------------|
| Claude Haiku 4.5 | 200K | $1 / $5 |
| Gemini 2.5 Flash | 1M | $0.30 / $2.50 |
| DeepSeek-chat | 128K | $0.28 / $0.42 |

## Installation

### With npx skills (Vercel)
```bash
npx skills add YOUR_USERNAME/ai-model-selector
```

### With skild
```bash
skild install YOUR_USERNAME/ai-model-selector
```

### With Claude Code
```bash
/plugin marketplace add YOUR_USERNAME/ai-model-selector
```

### Manual
Copy the folder to:
- Claude Code: ~/.claude/skills/ai-model-selector/
- Cursor: .cursor/skills/ai-model-selector/
- Codex: ~/.codex/skills/ai-model-selector/

## Example Usage

When you ask your AI agent:
- "What model should I use for coding?"
- "How do I call the GPT-5.2 API?"
- "Set up embeddings for RAG"

The agent will reference these files to give accurate, up-to-date answers.

## Models Covered

### Chat/Completion
- **OpenAI**: GPT-5.2, GPT-5.2 Pro, GPT-5.2-Codex, GPT-5.1, GPT-5, GPT-4.1, o3, o4-mini
- **Anthropic**: Claude Opus 4.5, Sonnet 4.5, Haiku 4.5
- **Google**: Gemini 3 Pro, Gemini 2.5 Pro, Gemini 2.5 Flash
- **DeepSeek**: V3.2 (deepseek-chat), R1 (deepseek-reasoner)

### Embeddings
- Voyage: voyage-3.5, voyage-3-large, voyage-code-3, voyage-law-2, voyage-finance-2
- OpenAI: text-embedding-3-large, text-embedding-3-small
- Google: gemini-embedding-001

## Critical Gotcha

GPT-5.x and o-series models do NOT support temperature, top_p, or max_tokens. Use:

```javascript
{
  "model": "gpt-5.2",
  "reasoning": { "effort": "medium" },
  "max_completion_tokens": 4096
}
```

## Contributing

PRs welcome! Model specs change frequently - please include sources when updating.

## License

MIT

## Last Updated

January 28, 2026

Sources: Official documentation from OpenAI, Anthropic, Google, DeepSeek, Voyage AI
