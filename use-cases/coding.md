# Coding Use Case Guide

## Model Recommendations by Task

| Task | Best Choice | Alternative | Budget |
|------|-------------|-------------|--------|
| **Agentic coding** | Claude Sonnet 4.5 | GPT-5.1-Codex | DeepSeek V3 |
| **Code review** | Claude Opus 4.5 | GPT-4.1 | DeepSeek V3 |
| **Quick edits** | Claude Haiku 4.5 | GPT-4.1-mini | DeepSeek Chat |
| **Algorithm design** | o3 | Claude Opus 4.5 | DeepSeek R1 |
| **Debugging** | Claude Sonnet 4.5 | GPT-4o | DeepSeek V3 |
| **Code completion** | Claude Haiku 4.5 | GPT-4.1-nano | Codestral |

## Benchmark Performance (SWE-bench Verified)

| Model | Score | Notes |
|-------|-------|-------|
| Claude Opus 4.5 | 80.9% | State-of-the-art |
| GPT-5.1-Codex-Max | 77.9% | Best OpenAI |
| Claude Sonnet 4.5 | 77.2% | Best value |
| Claude Haiku 4.5 | 73.3% | Fast + capable |
| DeepSeek V3.1 | ~71% | Budget champion |
| Gemini 2.5 Pro | 63.8% | With custom agent |

## Configuration by Task

### Agentic Coding (Claude Code, Cursor, etc.)

```python
# Claude Sonnet 4.5 - recommended for most agentic work
config = {
    "model": "claude-sonnet-4-5-20250929",
    "max_tokens": 16000,  # Allow long outputs for full files
    "temperature": 0,     # Deterministic for code
}

# For complex multi-file changes
config_extended = {
    "model": "claude-sonnet-4-5-20250929",
    "max_tokens": 64000,
    "thinking": {
        "type": "enabled",
        "budget_tokens": 10000
    }
}
```

### Code Review

```python
# Claude Opus 4.5 for thorough reviews
config = {
    "model": "claude-opus-4-5-20251101",
    "max_tokens": 8000,
    "temperature": 0.2,  # Slight variation for insights
    "effort": "high"     # Maximum attention
}

# System prompt
REVIEW_PROMPT = """You are a senior software engineer reviewing code.
Focus on:
1. Logic errors and edge cases
2. Security vulnerabilities
3. Performance issues
4. Code style and readability
5. Test coverage gaps

Be specific and provide code examples for fixes."""
```

### Quick Edits / Completions

```python
# Claude Haiku 4.5 for speed
config = {
    "model": "claude-haiku-4-5-20251001",
    "max_tokens": 2000,
    "temperature": 0,
}

# GPT-4.1-nano for highest throughput
config_openai = {
    "model": "gpt-4.1-nano",
    "max_tokens": 2000,
    "temperature": 0,
}
```

### Algorithm Design

```python
# OpenAI o3 for complex algorithms
config = {
    "model": "o3",
    "max_completion_tokens": 16000,
    "reasoning": {"effort": "high"},
}
# Note: No temperature support

# Claude with extended thinking
config_claude = {
    "model": "claude-opus-4-5-20251101",
    "max_tokens": 16000,
    "thinking": {
        "type": "enabled",
        "budget_tokens": 20000
    }
}
```

## Language-Specific Notes

### Python
- All models perform well
- Claude excels at type hints and modern Python
- DeepSeek V3 surprisingly good for the price

### JavaScript/TypeScript
- Claude Sonnet 4.5 handles complex TS types well
- GPT-4.1 good for React patterns
- All support modern ESM syntax

### Rust
- Claude models lead on SWE-bench Multilingual
- Ownership/borrow checker explanations are strong
- Use extended thinking for complex lifetime issues

### Go
- All major models handle well
- Claude good at idiomatic Go patterns

### Low-level (C, C++, Assembly)
- Claude Opus for complex memory management
- o3 for optimization puzzles
- Be explicit about target platform

## Prompting Tips

### 1. Be Explicit About Context

```python
CONTEXT_PROMPT = """
Project: {project_name}
Language: {language} {version}
Framework: {framework}
Build tool: {build_tool}
Style guide: {style_guide}
"""
```

### 2. Provide Examples

```python
EXAMPLE_PROMPT = """
Follow this pattern for error handling:

```python
def process_data(data: Data) -> Result[ProcessedData, Error]:
    if not data.is_valid():
        return Err(ValidationError("Invalid data"))
    
    try:
        result = transform(data)
        return Ok(result)
    except TransformError as e:
        return Err(e)
```
"""
```

### 3. Request Specific Output

```python
OUTPUT_PROMPT = """
Respond with:
1. Brief explanation of the approach (2-3 sentences)
2. Complete, runnable code
3. Example usage
4. Any edge cases to consider

Do not include markdown code fences in your response.
"""
```

## Cost Optimization

### 1. Use the Right Model for the Task

```python
def select_coding_model(task_type: str, complexity: str):
    if task_type == "completion" and complexity == "low":
        return "claude-haiku-4-5-20251001"  # Fast, cheap
    elif task_type == "review" and complexity == "high":
        return "claude-opus-4-5-20251101"   # Thorough
    else:
        return "claude-sonnet-4-5-20250929" # Default balanced
```

### 2. Cache Common Operations

```python
# Use prompt caching for repeated system prompts
system_prompt = {
    "type": "text",
    "text": LARGE_CODEBASE_CONTEXT,
    "cache_control": {"type": "ephemeral"}
}
```

### 3. Batch Similar Requests

```python
# Review multiple files in one request
files_to_review = [file1_content, file2_content, file3_content]
prompt = f"Review these {len(files_to_review)} files:\n\n" + \
         "\n---\n".join(files_to_review)
```

## Tool Configuration

### Claude Code

```yaml
# CLAUDE.md
Model: claude-sonnet-4-5-20250929
Temperature: 0
Max tokens: 16000

# For complex refactors
Extended thinking: enabled
Thinking budget: 10000
```

### Cursor

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "temperature": 0,
  "maxTokens": 8000
}
```

### Continue.dev

```json
{
  "models": [
    {
      "title": "Claude Sonnet",
      "provider": "anthropic",
      "model": "claude-sonnet-4-5-20250929",
      "apiKey": "..."
    }
  ]
}
```
