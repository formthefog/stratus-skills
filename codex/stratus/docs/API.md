# Stratus API Reference

Complete API documentation for both Stratus endpoints.

## Base URL

```
http://212.115.124.137:8000
```

## Authentication

All requests require Bearer token authentication:

```
Authorization: Bearer stratus_sk_live_YOUR_KEY_HERE
```

**Format validation only** - any `stratus_sk_test_*` or `stratus_sk_live_*` accepted (testing mode).

---

## Endpoints

### 1. Chat Completions (OpenAI-Compatible)

**Endpoint:** `POST /v1/chat/completions`

**Purpose:** Reasoning queries with Stratus world model + LLM execution

**Request:**
```json
{
  "model": "stratus-x1ac-{size}-{llm}",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant"},
    {"role": "user", "content": "Count r in strawberry"}
  ],
  "stream": false,
  "temperature": 0.7,
  "max_tokens": 1000
}
```

**Response:**
```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1706889600,
  "model": "stratus-x1ac-small-claude-sonnet-4-5",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "3"
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 12,
    "completion_tokens": 1,
    "total_tokens": 13
  },
  "stratus": {
    "stratus_model": "x1ac-small",
    "execution_llm": "claude-sonnet-4-5",
    "action_sequence": ["analyze", "count", "respond"],
    "confidence": 0.95,
    "planning_time_ms": 0.004,
    "execution_time_ms": 1234.5,
    "token_reduction": 24.3
  }
}
```

**Model naming:** `stratus-x1ac-{size}-{llm}`
- Sizes: debug, small, medium
- LLMs: claude-sonnet-4-5, claude-opus-4, gpt-4o, etc.

---

### 2. Rollout (Multi-Step Planning)

**Endpoint:** `POST /v1/rollout`

**Purpose:** Multi-step action prediction and validation

#### Request Format 1: Goal-Based Planning

```json
{
  "goal": "string (required)",
  "max_steps": 5,
  "return_intermediate": true,
  "initial_state": "optional state description"
}
```

#### Request Format 2: Explicit Action Sequence

```json
{
  "goal": "string (required)",
  "actions": [17, 10, 28],
  "return_intermediate": true
}
```

**Response:**
```json
{
  "id": "stratus-rollout-95e45106220c",
  "object": "rollout.prediction",
  "created": 1706889600,
  "goal": "Find and book hotel in San Francisco",
  "initial_state": "User on homepage",
  "action_sequence": [
    {
      "step": 1,
      "action_id": 17,
      "action_name": "search",
      "action_category": "retrieval"
    },
    {
      "step": 2,
      "action_id": 10,
      "action_name": "select",
      "action_category": "navigation"
    },
    {
      "step": 3,
      "action_id": 28,
      "action_name": "read",
      "action_category": "retrieval"
    }
  ],
  "predictions": [
    {
      "step": 1,
      "action": {
        "step": 1,
        "action_id": 17,
        "action_name": "search",
        "action_category": "retrieval"
      },
      "current_state": {
        "step": 0,
        "magnitude": 18.2,
        "confidence": "High"
      },
      "predicted_state": {
        "step": 1,
        "magnitude": 16.5,
        "confidence": "High"
      },
      "state_change": 7.8,
      "interpretation": "Query narrows search space"
    }
  ],
  "summary": {
    "total_steps": 3,
    "initial_magnitude": 18.2,
    "final_magnitude": 10.1,
    "total_state_change": 23.4,
    "outcome": "Goal likely achieved (large cumulative change)",
    "action_path": ["search", "select", "read"]
  }
}
```

---

## Response Fields

### Stratus Metadata (Chat Completions)

| Field | Type | Description |
|-------|------|-------------|
| `stratus_model` | string | Stratus model size (x1ac-small, x1ac-medium) |
| `execution_llm` | string | LLM used for execution |
| `action_sequence` | string[] | Actions predicted by world model |
| `confidence` | number | Prediction confidence (0-1) |
| `planning_time_ms` | number | Stratus planning time |
| `execution_time_ms` | number | LLM execution time |
| `token_reduction` | number | Token reduction factor |

### State Info (Rollout)

| Field | Type | Description |
|-------|------|-------------|
| `magnitude` | number | State complexity indicator |
| `confidence` | string | "High", "Medium", or "Low" |

**State magnitude interpretation:**
- < 10: Low complexity
- 10-15: Medium complexity
- 15-20: High complexity (typical web tasks)
- > 20: Very high complexity

### State Change (Rollout)

| Field | Type | Description |
|-------|------|-------------|
| `state_change` | number | Magnitude of state transition |

**State change interpretation:**
- < 5: Minor transition
- 5-10: Moderate transition
- 10-15: Significant transition
- > 15: Major transition

### Outcome (Rollout Summary)

| Outcome | Meaning | Typical Total Change |
|---------|---------|---------------------|
| "Goal likely achieved" | Strong prediction | >20 |
| "Significant progress" | Moderate prediction | 10-20 |
| "Minimal progress" | Weak prediction | <10 |

---

## Action Categories

| Category | Examples | Use Case |
|----------|----------|----------|
| **retrieval** | search, read, extract, scan | Information gathering |
| **navigation** | select, click, navigate, scroll | Moving through UI |
| **interaction** | type, submit, edit, delete | Modifying state |
| **system** | pad, unk, start | Special tokens |

---

## Error Responses

### 401 Unauthorized

```json
{
  "error": {
    "message": "Invalid API key format",
    "type": "authentication_error",
    "code": "invalid_api_key"
  }
}
```

**Fix:** Verify API key starts with `stratus_sk_live_` or `stratus_sk_test_`

### 422 Unprocessable Entity

```json
{
  "error": {
    "message": "Missing required field: goal",
    "type": "validation_error",
    "code": "missing_field"
  }
}
```

**Fix:** Check request body includes all required fields

### 500 Internal Server Error

```json
{
  "error": {
    "message": "Model prediction failed",
    "type": "server_error",
    "code": "prediction_error"
  }
}
```

**Fix:** Check server logs, retry request

---

## Rate Limits

**Current limits (testing mode):**
- No enforced rate limits
- Recommended: <100 req/s per client

**Production limits (TBD):**
- Will be based on tier/plan
- Contact Formation for details

---

## Known Issues

| Issue | Workaround |
|-------|-----------|
| `return_intermediate: false` crashes | Always use `true` |
| No real auth validation | Testing mode only |
| Action IDs not documented | Reference action catalog (TBD) |

---

## Examples

### Python (requests)

```python
import requests

response = requests.post(
    "http://212.115.124.137:8000/v1/chat/completions",
    headers={
        "Authorization": "Bearer stratus_sk_live_...",
        "Content-Type": "application/json"
    },
    json={
        "model": "stratus-x1ac-small-gpt-4o",
        "messages": [
            {"role": "user", "content": "Count r in strawberry"}
        ]
    }
)

result = response.json()
print(result['choices'][0]['message']['content'])
```

### Python (OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://212.115.124.137:8000/v1",
    api_key="stratus_sk_live_..."
)

response = client.chat.completions.create(
    model="stratus-x1ac-small-claude-sonnet-4-5",
    messages=[
        {"role": "user", "content": "Is 127 prime?"}
    ]
)

print(response.choices[0].message.content)
```

### JavaScript/TypeScript

```typescript
const response = await fetch('http://212.115.124.137:8000/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer stratus_sk_live_...',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    model: 'stratus-x1ac-small-gpt-4o',
    messages: [
      { role: 'user', content: 'Count r in strawberry' }
    ]
  })
});

const result = await response.json();
console.log(result.choices[0].message.content);
```

### cURL

```bash
curl -X POST http://212.115.124.137:8000/v1/chat/completions \
  -H "Authorization: Bearer stratus_sk_live_..." \
  -H "Content-Type: application/json" \
  -d '{
    "model": "stratus-x1ac-small-claude-sonnet-4-5",
    "messages": [
      {"role": "user", "content": "Is 8191 prime?"}
    ]
  }'
```

---

**For more:**
- Integration guide: `INTEGRATION.md`
- Architecture: `ARCHITECTURE.md`
- Main skill: `../SKILL.md`
