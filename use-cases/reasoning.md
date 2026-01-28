# Reasoning Use Case Guide

## Model Recommendations

| Task | Best Choice | Alternative | Budget |
|------|-------------|-------------|--------|
| **Math competitions** | o3 | o4-mini | DeepSeek R1 |
| **Scientific analysis** | o3 | Claude Opus 4.5 | Gemini 2.5 Pro |
| **Logic puzzles** | o3 | Claude + thinking | DeepSeek R1 |
| **Multi-step planning** | Claude Opus 4.5 | GPT-5 | DeepSeek R1 |
| **Research synthesis** | Claude Opus 4.5 | Gemini 2.5 Pro | DeepSeek V3 |
| **Legal reasoning** | Claude Opus 4.5 | o3 | GPT-4.1 |

## Reasoning Model Comparison

| Model | AIME 2025 | GPQA | Math | Approach |
|-------|-----------|------|------|----------|
| **o3** | ~90% | 87.7% | 96.7% | Hidden CoT |
| **o4-mini** | 93.4% | - | - | Hidden CoT |
| **Claude Opus 4.5** | - | - | - | Extended thinking |
| **DeepSeek R1** | 79.8% | 71.5% | 97.3% | Visible CoT |
| **Gemini 2.5 Pro** | - | - | - | Thinking mode |

## Configuration Patterns

### OpenAI Reasoning Models (o3, o4-mini)

```python
from openai import OpenAI

client = OpenAI()

# ⚠️ CRITICAL: No temperature, use reasoning effort
response = client.responses.create(
    model="o3",
    reasoning={"effort": "high"},  # low, medium, high
    input=[
        {"role": "user", "content": "Prove that √2 is irrational"}
    ]
)

print(response.output_text)

# Check reasoning token usage
print(f"Reasoning tokens: {response.usage.reasoning_tokens}")
print(f"Output tokens: {response.usage.completion_tokens}")
```

### Claude Extended Thinking

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-5-20251101",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 20000  # Allow extensive reasoning
    },
    messages=[{
        "role": "user",
        "content": "Analyze the implications of Gödel's incompleteness theorems"
    }]
)

# Access both thinking and response
for block in response.content:
    if block.type == "thinking":
        print(f"THINKING:\n{block.thinking}\n")
    elif block.type == "text":
        print(f"RESPONSE:\n{block.text}")
```

### DeepSeek R1

```python
from openai import OpenAI

client = OpenAI(
    api_key="DEEPSEEK_KEY",
    base_url="https://api.deepseek.com"
)

response = client.chat.completions.create(
    model="deepseek-reasoner",
    messages=[{
        "role": "user",
        "content": "Solve: Find all integer solutions to x³ + y³ = z³"
    }],
    max_tokens=16000,  # R1 needs space for CoT
    stream=True
)

# Reasoning is visible in response
for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Gemini Thinking Mode

```python
import google.generativeai as genai

model = genai.GenerativeModel(
    model_name="gemini-2.5-pro",
    generation_config={
        "thinking_config": {
            "thinking_budget": 20000
        }
    }
)

response = model.generate_content(
    "Derive the Schwarzschild metric from first principles"
)
```

## Prompting for Reasoning

### 1. Explicit Step-by-Step

```python
MATH_PROMPT = """Solve this problem step by step.

Problem: {problem}

Instructions:
1. First, identify what we're solving for
2. List all given information
3. Determine which formulas/theorems apply
4. Show each calculation step
5. Verify your answer

Show all work."""
```

### 2. Encourage Self-Verification

```python
VERIFY_PROMPT = """Solve this problem, then verify your solution.

Problem: {problem}

After solving:
- Check your answer by substituting back
- Consider edge cases
- Identify any assumptions made
- Rate your confidence (1-10)"""
```

### 3. Multi-Perspective Analysis

```python
ANALYSIS_PROMPT = """Analyze this from multiple perspectives:

Topic: {topic}

Consider:
1. Historical context
2. Current state of knowledge
3. Competing theories/viewpoints
4. Evidence for and against
5. Implications and predictions
6. Areas of uncertainty

Synthesize into a balanced conclusion."""
```

## Task-Specific Configurations

### Mathematical Proofs

```python
config_proof = {
    "model": "o3",
    "reasoning": {"effort": "high"},
    # Proofs need maximum reasoning
}

# Or with Claude
config_proof_claude = {
    "model": "claude-opus-4-5-20251101",
    "thinking": {
        "type": "enabled",
        "budget_tokens": 30000  # Proofs need extensive thinking
    },
    "max_tokens": 8000
}
```

### Scientific Analysis

```python
config_science = {
    "model": "claude-opus-4-5-20251101",
    "max_tokens": 16000,
    "thinking": {"type": "enabled", "budget_tokens": 15000},
    "temperature": 0.3  # Slight variation for hypothesis generation
}
```

### Strategic Planning

```python
config_planning = {
    "model": "claude-opus-4-5-20251101",
    "max_tokens": 12000,
    "effort": "high",  # Opus 4.5 effort parameter
}

PLANNING_PROMPT = """Create a comprehensive plan for: {goal}

Structure:
1. Goal clarification and success metrics
2. Constraint analysis
3. Option generation (at least 3 approaches)
4. Risk assessment for each option
5. Recommended approach with rationale
6. Implementation timeline
7. Contingency plans"""
```

## Cost vs Quality Tradeoffs

### High-Stakes (use best models)
- Mathematical proofs
- Legal analysis
- Medical reasoning
- Financial modeling

```python
HIGH_STAKES = {
    "primary": "o3",
    "alternative": "claude-opus-4-5-20251101",
    "reasoning_effort": "high"
}
```

### Medium Complexity (balanced)
- Research summaries
- Technical analysis
- Strategic planning

```python
MEDIUM = {
    "primary": "claude-sonnet-4-5-20250929",
    "alternative": "gpt-5",
    "thinking_budget": 10000
}
```

### High Volume (cost-efficient)
- Batch analysis
- Initial screening
- Simple logical tasks

```python
VOLUME = {
    "primary": "deepseek-r1",  # Very cheap
    "alternative": "o4-mini",
    "fallback": "claude-haiku-4-5-20251001"
}
```

## Handling Reasoning Failures

### 1. Retry with Higher Effort

```python
def reason_with_retry(prompt, max_attempts=3):
    efforts = ["medium", "high", "high"]
    
    for i, effort in enumerate(efforts):
        response = client.responses.create(
            model="o3",
            reasoning={"effort": effort},
            input=[{"role": "user", "content": prompt}]
        )
        
        if validate_reasoning(response.output_text):
            return response
    
    return None
```

### 2. Switch Models

```python
def reason_with_fallback(prompt):
    # Try o3 first
    response = try_model("o3", prompt)
    if validate(response):
        return response
    
    # Fall back to Claude with thinking
    response = try_model("claude-opus-4-5", prompt, thinking=True)
    if validate(response):
        return response
    
    # Last resort: DeepSeek R1 (visible reasoning for debugging)
    return try_model("deepseek-r1", prompt)
```

### 3. Decompose Complex Problems

```python
def solve_complex(problem):
    # Break into subproblems
    decomposition = llm.chat("Break this into simpler subproblems:\n" + problem)
    
    subproblems = parse_subproblems(decomposition)
    solutions = []
    
    for sub in subproblems:
        solution = llm.chat(f"Solve: {sub}", thinking=True)
        solutions.append(solution)
    
    # Synthesize
    final = llm.chat(
        f"Combine these solutions into a final answer:\n{solutions}"
    )
    
    return final
```
