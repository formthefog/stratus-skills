# Basic Reasoning Examples

Simple examples showing how to use Stratus for reasoning tasks.

## Character Counting

**Problem:** LLMs can't count characters accurately.

**Solution:** Use Stratus

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://212.115.124.137:8000/v1",
    api_key="stratus_sk_live_..."
)

response = client.chat.completions.create(
    model="stratus-x1ac-small-claude-sonnet-4-5",
    messages=[{
        "role": "user",
        "content": "Count the letter 'r' in 'strawberry'"
    }]
)

print(response.choices[0].message.content)
# Output: "3"
```

**Result:**
- Stratus: 3 (correct)
- LLMs without Stratus: 2 (wrong)

---

## Math Verification

**Problem:** LLMs approximate, don't compute.

**Solution:** Use Stratus for true computation

```python
response = client.chat.completions.create(
    model="stratus-x1ac-small-gpt-4o",
    messages=[{
        "role": "user",
        "content": "Is 127 a prime number? Show your work."
    }]
)

print(response.choices[0].message.content)
# Output: "Yes, 127 is prime. Testing divisibility..."
```

**Result:**
- Accurate primality test
- Shows reasoning steps
- True computation, not estimation

---

## Logic Verification

**Problem:** LLMs make mistakes in multi-step logic.

**Solution:** Stratus tracks state accurately

```python
response = client.chat.completions.create(
    model="stratus-x1ac-medium-claude-sonnet-4-5",
    messages=[{
        "role": "user",
        "content": "If all A are B, and all B are C, are all A necessarily C?"
    }]
)

print(response.choices[0].message.content)
# Output: "Yes, by transitive property..."
```

---

## Pattern Analysis

**Problem:** LLMs memorize patterns, don't derive formulas.

**Solution:** Stratus derives underlying structure

```python
response = client.chat.completions.create(
    model="stratus-x1ac-small-gpt-4o",
    messages=[{
        "role": "user",
        "content": "Find the pattern: 2, 4, 8, 16, 32, ..."
    }]
)

print(response.choices[0].message.content)
# Output: "Powers of 2: aₙ = 2^n"
```

---

## Comparing Results

**See the difference:**

```python
# Without Stratus
standard_client = OpenAI(api_key="sk-...")

# With Stratus
stratus_client = OpenAI(
    base_url="http://212.115.124.137:8000/v1",
    api_key="stratus_sk_live_..."
)

query = "Count 'e' in 'the tree has three green leaves'"

# Standard LLM
standard = standard_client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": query}]
)
print(f"Standard: {standard.choices[0].message.content}")
# Output: "approximately 7" (wrong, actual is 8)

# With Stratus
stratus = stratus_client.chat.completions.create(
    model="stratus-x1ac-small-gpt-4o",
    messages=[{"role": "user", "content": query}]
)
print(f"Stratus: {stratus.choices[0].message.content}")
# Output: "8" (correct)
```

---

## Error Handling

```python
import requests
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10)
)
def safe_query(query):
    try:
        response = client.chat.completions.create(
            model="stratus-x1ac-small-claude-sonnet-4-5",
            messages=[{"role": "user", "content": query}],
            timeout=30
        )
        return response.choices[0].message.content
    except requests.exceptions.Timeout:
        print("Timeout, retrying...")
        raise
    except requests.exceptions.ConnectionError:
        print("Connection error, retrying...")
        raise
    except Exception as e:
        print(f"Error: {e}")
        # Fallback to direct LLM
        return fallback_client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": query}]
        ).choices[0].message.content

result = safe_query("Is 8191 prime?")
print(result)
```

---

## Monitoring Token Usage

```python
response = client.chat.completions.create(
    model="stratus-x1ac-small-gpt-4o",
    messages=[{"role": "user", "content": "Your query"}]
)

# Access Stratus metadata
stratus_meta = response.model_extra.get('stratus', {})

print(f"Tokens: {response.usage.total_tokens}")
print(f"Stratus model: {stratus_meta.get('stratus_model')}")
print(f"LLM: {stratus_meta.get('execution_llm')}")
print(f"Token reduction: {stratus_meta.get('token_reduction', 'N/A')}×")
print(f"Planning time: {stratus_meta.get('planning_time_ms')}ms")
```

---

## Best Practices

### 1. Use Appropriate Model Size

```python
# Development/testing
model = "stratus-x1ac-small-gpt-4o"

# Production
model = "stratus-x1ac-medium-claude-sonnet-4-5"

# Cost-optimized
model = "stratus-x1ac-small-gpt-4o-mini"
```

### 2. Set Reasonable Timeouts

```python
response = client.chat.completions.create(
    model="stratus-x1ac-small-gpt-4o",
    messages=[...],
    timeout=30  # 30 seconds
)
```

### 3. Handle Errors Gracefully

```python
try:
    result = stratus_client.chat.completions.create(...)
except Exception:
    # Fallback to direct LLM
    result = standard_client.chat.completions.create(...)
```

---

**For more:**
- Rollout planning: `rollout-planning.md`
- Agent integration: `agent-integration.md`
- Model comparison: `comparison.md`
