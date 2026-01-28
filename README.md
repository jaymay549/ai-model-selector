# AI Model Selector Skill

A comprehensive guide to AI model selection and API implementation for AI coding agents.

## What's Included

| File | Description |
|------|-------------|
| `SKILL.md` | Main entry point with decision tree |
| `models/openai.md` | GPT-4.1, GPT-5, o-series specs |
| `models/anthropic.md` | Claude 4.5 family |
| `models/google.md` | Gemini 2.5 Pro/Flash |
| `models/deepseek.md` | DeepSeek V3, R1 |
| `models/embeddings.md` | Voyage, OpenAI, Cohere embeddings |
| `use-cases/coding.md` | Best models for coding tasks |
| `use-cases/reasoning.md` | Math, science, planning |
| `use-cases/rag.md` | RAG pipeline guidance |

## Key Features

- **Decision tree** for quick model selection
- **Exact model strings** and API parameters
- **Critical gotchas** (e.g., o-series doesn't support temperature)
- **Code snippets** for each provider
- **Pricing** and context window limits
- **Use-case specific** configurations

## Installation

### With npx skills (Vercel)
```bash
npx skills add jaymay549/ai-model-selector
```

### With skild
```bash
skild install jaymay549/ai-model-selector
```

### With Claude Code
```bash
/plugin marketplace add jaymay549/ai-model-selector
```

### Manual
Copy the folder to:
- Claude Code: `~/.claude/skills/ai-model-selector/`
- Cursor: `.cursor/skills/ai-model-selector/`
- Codex: `~/.codex/skills/ai-model-selector/`

## Example Usage

When you ask your AI agent:
- "What model should I use for coding?"
- "How do I call the o3 API?"
- "Set up embeddings for RAG"

The agent will reference these files to give accurate, up-to-date answers.

## Models Covered

### Chat/Completion
- **OpenAI**: GPT-4.1, GPT-4.1-mini, GPT-4.1-nano, GPT-4o, GPT-5, o1, o3, o4-mini
- **Anthropic**: Claude Opus 4.5, Sonnet 4.5, Haiku 4.5
- **Google**: Gemini 2.5 Pro, Gemini 2.5 Flash
- **DeepSeek**: V3, V3.1, R1, R1-0528

### Embeddings
- OpenAI text-embedding-3-small/large
- Voyage 3-large, Code 3, Law 2, Finance 2
- Cohere embed-v3
- Gemini embedding-001

## Contributing

PRs welcome! Model specs change frequently - please include sources when updating.

## License

MIT
