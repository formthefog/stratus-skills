# Stratus X1 Integration Guide

How to integrate Stratus into your agent stack.

## Quick Start (2 Minutes)

### Option 1: OpenAI-Compatible (Recommended)

**Change 2 lines of code:**

```python
from openai import OpenAI

# Before
client = OpenAI(api_key="sk-...")

# After
client = OpenAI(
    base_url="http://212.115.124.137:8000/v1",  # ← Change 1
    api_key="stratus_sk_live_..."                # ← Change 2
)

# Same API, now with Stratus world model
response = client.chat.completions.create(
    model="stratus-x1ac-small-claude-sonnet-4-5",  # Combined model
    messages=[{"role": "user", "content": "Your query"}]
)
```

**That's it. Your agent now has:**
- 20-30× token reduction
- Multi-step planning capability
- Hallucination detection
- State prediction

### Option 2: Direct Model API

For advanced users who want direct access:

```python
import requests

# Rollout endpoint for planning
response = requests.post(
    "http://212.115.124.137:8000/v1/rollout",
    headers={"Authorization": "Bearer stratus_sk_live_..."},
    json={
        "goal": "Find and book hotel in SF",
        "max_steps": 5,
        "return_intermediate": True
    }
)

plan = response.json()
actions = plan['summary']['action_path']
# ['search', 'select', 'read', 'type', 'submit']
```

---

## Environment Setup

### Required Variables

```bash
# .env
STRATUS_API_KEY=stratus_sk_live_YOUR_KEY_HERE
STRATUS_API_URL=http://212.115.124.137:8000
```

### Optional Variables

```bash
# For model comparison
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...

# For debugging
STRATUS_DEBUG=1
STRATUS_TIMEOUT=30
```

---

## Model Naming Convention

**Format:** `stratus-x1ac-{size}-{llm}`

**Examples:**
- `stratus-x1ac-small-claude-sonnet-4-5` (recommended)
- `stratus-x1ac-small-gpt-4o`
- `stratus-x1ac-medium-claude-opus-4`

**Sizes:**
- `debug` (16M) - Fast iteration
- `small` (125M) - Development/production
- `medium` (350M) - Maximum accuracy

**LLMs:**
- `claude-sonnet-4-5` (recommended)
- `claude-opus-4`
- `gpt-4o`
- `gpt-4-turbo`
- `gpt-4o-mini`

---

## Integration Patterns

### Pattern 1: Drop-in Replacement

Replace OpenAI client, everything else stays the same:

```python
# Before
from openai import OpenAI
client = OpenAI(api_key="sk-...")

# After
client = OpenAI(
    base_url="http://212.115.124.137:8000/v1",
    api_key="stratus_sk_live_..."
)

# Rest of code unchanged
```

**Benefits:**
- Zero code changes
- Immediate 20-30× token reduction
- All existing error handling works
- Compatible with all OpenAI libraries

### Pattern 2: Hybrid (Stratus + Direct LLM)

Use Stratus for agent tasks, direct LLM for chat:

```python
# Stratus client for agent tasks
stratus_client = OpenAI(
    base_url="http://212.115.124.137:8000/v1",
    api_key="stratus_sk_live_..."
)

# Direct LLM for chat
llm_client = OpenAI(api_key="sk-...")

# Agent tasks → Stratus
agent_response = stratus_client.chat.completions.create(
    model="stratus-x1ac-small-gpt-4o",
    messages=agent_messages
)

# User chat → Direct LLM
chat_response = llm_client.chat.completions.create(
    model="gpt-4o",
    messages=chat_messages
)
```

### Pattern 3: Planning Before Execution

Use rollout endpoint to validate plans:

```python
def execute_task(goal):
    # 1. Plan with Stratus
    plan = requests.post(
        "http://212.115.124.137:8000/v1/rollout",
        headers={"Authorization": f"Bearer {api_key}"},
        json={"goal": goal, "max_steps": 5}
    ).json()

    # 2. Validate plan
    if "Goal likely achieved" not in plan['summary']['outcome']:
        # Replan with more steps
        return execute_task_extended(goal)

    # 3. Execute validated plan
    for action in plan['summary']['action_path']:
        execute_action(action)
```

---

## Agent Framework Integration

### LangChain

```python
from langchain.llms import OpenAI

llm = OpenAI(
    openai_api_base="http://212.115.124.137:8000/v1",
    openai_api_key="stratus_sk_live_...",
    model_name="stratus-x1ac-small-gpt-4o"
)

# Use in agent
from langchain.agents import initialize_agent, Tool

agent = initialize_agent(
    tools=[...],
    llm=llm,
    agent="zero-shot-react-description"
)
```

### AutoGen

```python
from autogen import AssistantAgent, UserProxyAgent

assistant = AssistantAgent(
    name="assistant",
    llm_config={
        "config_list": [{
            "model": "stratus-x1ac-small-claude-sonnet-4-5",
            "base_url": "http://212.115.124.137:8000/v1",
            "api_key": "stratus_sk_live_..."
        }]
    }
)
```

### Custom Agent Loop

```python
class StratusAgent:
    def __init__(self):
        self.client = OpenAI(
            base_url="http://212.115.124.137:8000/v1",
            api_key="stratus_sk_live_..."
        )
        self.model = "stratus-x1ac-small-claude-sonnet-4-5"

    def run(self, goal):
        # Plan
        plan = self.plan(goal)

        # Execute
        for action in plan['summary']['action_path']:
            result = self.execute(action)

            # Validate
            if not self.validate(result, plan):
                # Replan
                plan = self.plan(goal, context=result)

        return result
```

---

## Error Handling

### Connection Errors

```python
import requests
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10)
)
def call_stratus(query):
    try:
        response = client.chat.completions.create(
            model="stratus-x1ac-small-claude-sonnet-4-5",
            messages=[{"role": "user", "content": query}]
        )
        return response
    except requests.exceptions.ConnectionError:
        print("Stratus API unreachable, retrying...")
        raise
    except requests.exceptions.Timeout:
        print("Request timed out, retrying...")
        raise
```

### Fallback to Direct LLM

```python
def query_with_fallback(query):
    try:
        # Try Stratus first
        return stratus_client.chat.completions.create(
            model="stratus-x1ac-small-gpt-4o",
            messages=[{"role": "user", "content": query}]
        )
    except Exception as e:
        print(f"Stratus failed: {e}, falling back to direct LLM")
        # Fallback to direct LLM
        return llm_client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": query}]
        )
```

---

## Performance Optimization

### Batching

```python
# Batch multiple predictions
predictions = []
for goal in goals:
    predictions.append(
        requests.post(..., json={"goal": goal})
    )

# Process in parallel
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor(max_workers=10) as executor:
    results = list(executor.map(call_stratus, queries))
```

### Caching

```python
from functools import lru_cache

@lru_cache(maxsize=1000)
def predict_state(state_hash, action):
    response = requests.post(..., json={
        "goal": action,
        "initial_state": state_hash
    })
    return response.json()
```

---

## Monitoring

### Track Token Reduction

```python
def track_usage(response):
    stratus_meta = response.get('stratus', {})

    print(f"Stratus model: {stratus_meta['stratus_model']}")
    print(f"LLM: {stratus_meta['execution_llm']}")
    print(f"Tokens: {response['usage']['total_tokens']}")
    print(f"Planning time: {stratus_meta['planning_time_ms']}ms")
    print(f"Execution time: {stratus_meta['execution_time_ms']}ms")
```

### Log Hallucination Detection

```python
def check_hallucination(predicted_state, actual_state):
    # Compare predicted vs actual
    distance = embedding_distance(predicted_state, actual_state)

    if distance > threshold:
        log_hallucination({
            "predicted": predicted_state,
            "actual": actual_state,
            "distance": distance
        })
```

---

## Production Deployment

### Docker

```dockerfile
FROM python:3.11-slim

# Install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Set environment
ENV STRATUS_API_KEY=stratus_sk_live_...
ENV STRATUS_API_URL=http://212.115.124.137:8000

# Run agent
CMD ["python", "agent.py"]
```

### Kubernetes

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: stratus-credentials
stringData:
  api-key: stratus_sk_live_...

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: agent
spec:
  template:
    spec:
      containers:
      - name: agent
        image: your-agent:latest
        env:
        - name: STRATUS_API_KEY
          valueFrom:
            secretKeyRef:
              name: stratus-credentials
              key: api-key
        - name: STRATUS_API_URL
          value: "http://212.115.124.137:8000"
```

---

## Best Practices

### 1. Use Stratus for Agent Tasks

✅ DO use Stratus for:
- Multi-step workflows
- Tool-using agents
- State-dependent tasks
- Long-horizon planning

❌ DON'T use Stratus for:
- Simple chat
- Creative writing
- Single-turn Q&A

### 2. Validate Plans Before Execution

```python
# Good
plan = get_plan(goal)
if validate_plan(plan):
    execute(plan)

# Bad
execute(get_plan(goal))  # No validation
```

### 3. Handle Errors Gracefully

```python
# Good
try:
    result = stratus_client.chat.completions.create(...)
except Exception:
    result = fallback_client.chat.completions.create(...)

# Bad
result = stratus_client.chat.completions.create(...)  # No error handling
```

### 4. Monitor Token Usage

Track before/after to verify token reduction is working.

### 5. Start Small

Begin with one agent task, verify it works, then scale.

---

## Troubleshooting

**Connection refused:**
```bash
# Check server status
curl http://212.115.124.137:8000/health
```

**Invalid API key:**
```bash
# Verify format
echo $STRATUS_API_KEY | grep "stratus_sk_live_"
```

**Slow responses:**
```bash
# Check latency
time curl -X POST http://212.115.124.137:8000/v1/chat/completions \
  -H "Authorization: Bearer $STRATUS_API_KEY" \
  -d '{"model": "stratus-x1ac-small-gpt-4o", "messages": [...]}'
```

---

## Next Steps

1. Set environment variables
2. Run test query: `/stratus test`
3. Try a puzzle: `/stratus puzzle 1`
4. Integrate into your agent
5. Monitor token reduction
6. Scale to production

---

**For more:**
- Examples: `../examples/`
- Architecture: `ARCHITECTURE.md`
- Benchmarks: `BENCHMARKS.md`
- Main skill: `../SKILL.md`
