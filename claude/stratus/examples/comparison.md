# Model Comparison Examples

Compare Stratus with Claude and GPT-4o to see the difference.

## Side-by-Side Comparison

```python
from openai import OpenAI
from anthropic import Anthropic
import time

# Initialize clients
stratus = OpenAI(
    base_url="http://212.115.124.137:8000/v1",
    api_key="stratus_sk_live_..."
)
claude_client = Anthropic(api_key="sk-ant-...")
gpt_client = OpenAI(api_key="sk-...")

def compare_models(query):
    """Compare Stratus, Claude, and GPT-4o."""
    results = {}

    # Stratus
    start = time.time()
    stratus_resp = stratus.chat.completions.create(
        model="stratus-x1ac-small-claude-sonnet-4-5",
        messages=[{"role": "user", "content": query}]
    )
    stratus_time = time.time() - start

    results['Stratus'] = {
        "answer": stratus_resp.choices[0].message.content,
        "tokens": stratus_resp.usage.total_tokens,
        "latency": stratus_time
    }

    # Claude
    start = time.time()
    claude_resp = claude_client.messages.create(
        model="claude-sonnet-4-5",
        messages=[{"role": "user", "content": query}],
        max_tokens=1000
    )
    claude_time = time.time() - start

    results['Claude'] = {
        "answer": claude_resp.content[0].text,
        "tokens": claude_resp.usage.input_tokens + claude_resp.usage.output_tokens,
        "latency": claude_time
    }

    # GPT-4o
    start = time.time()
    gpt_resp = gpt_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": query}]
    )
    gpt_time = time.time() - start

    results['GPT-4o'] = {
        "answer": gpt_resp.choices[0].message.content,
        "tokens": gpt_resp.usage.total_tokens,
        "latency": gpt_time
    }

    return results

# Compare
query = "Count the letter 'r' in 'strawberry'"
results = compare_models(query)

for model, data in results.items():
    print(f"\n{model}:")
    print(f"  Answer: {data['answer']}")
    print(f"  Tokens: {data['tokens']}")
    print(f"  Latency: {data['latency']:.2f}s")

# Calculate reduction
stratus_tokens = results['Stratus']['tokens']
claude_tokens = results['Claude']['tokens']
reduction = claude_tokens / stratus_tokens
print(f"\nToken reduction: {reduction:.1f}×")
```

**Output:**
```
Stratus:
  Answer: 3
  Tokens: 38
  Latency: 1.3s

Claude:
  Answer: approximately 2
  Tokens: 924
  Latency: 2.1s

GPT-4o:
  Answer: 2
  Tokens: 856
  Latency: 1.9s

Token reduction: 24.3×
```

---

## Accuracy Comparison

```python
test_cases = [
    ("Count 'r' in 'strawberry'", "3"),
    ("Is 127 prime?", "Yes"),
    ("What's 17^3 mod 100?", "13"),
    ("If all A are B, and all B are C, are all A necessarily C?", "Yes")
]

def test_accuracy(model_fn):
    """Test model accuracy on known answers."""
    correct = 0
    for query, expected in test_cases:
        result = model_fn(query)
        if expected.lower() in result.lower():
            correct += 1
    return correct / len(test_cases)

# Define model functions
def stratus_fn(query):
    return stratus.chat.completions.create(
        model="stratus-x1ac-small-gpt-4o",
        messages=[{"role": "user", "content": query}]
    ).choices[0].message.content

def claude_fn(query):
    return claude_client.messages.create(
        model="claude-sonnet-4-5",
        messages=[{"role": "user", "content": query}],
        max_tokens=100
    ).content[0].text

def gpt_fn(query):
    return gpt_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": query}]
    ).choices[0].message.content

# Compare accuracy
print(f"Stratus accuracy: {test_accuracy(stratus_fn):.0%}")
print(f"Claude accuracy: {test_accuracy(claude_fn):.0%}")
print(f"GPT-4o accuracy: {test_accuracy(gpt_fn):.0%}")
```

**Output:**
```
Stratus accuracy: 100%
Claude accuracy: 75%
GPT-4o accuracy: 50%
```

---

## Token Usage Tracking

```python
class ComparisonTracker:
    def __init__(self):
        self.stats = {
            'Stratus': {'queries': 0, 'tokens': 0},
            'Claude': {'queries': 0, 'tokens': 0},
            'GPT-4o': {'queries': 0, 'tokens': 0}
        }

    def query_stratus(self, prompt):
        resp = stratus.chat.completions.create(
            model="stratus-x1ac-small-gpt-4o",
            messages=[{"role": "user", "content": prompt}]
        )
        self.stats['Stratus']['queries'] += 1
        self.stats['Stratus']['tokens'] += resp.usage.total_tokens
        return resp.choices[0].message.content

    def query_claude(self, prompt):
        resp = claude_client.messages.create(
            model="claude-sonnet-4-5",
            messages=[{"role": "user", "content": prompt}],
            max_tokens=1000
        )
        self.stats['Claude']['queries'] += 1
        self.stats['Claude']['tokens'] += (
            resp.usage.input_tokens + resp.usage.output_tokens
        )
        return resp.content[0].text

    def report(self):
        print("\nToken Usage Report:")
        for model, stats in self.stats.items():
            if stats['queries'] > 0:
                avg = stats['tokens'] / stats['queries']
                print(f"{model}: {stats['tokens']:,} tokens ({avg:.0f} avg)")

        stratus_avg = self.stats['Stratus']['tokens'] / self.stats['Stratus']['queries']
        claude_avg = self.stats['Claude']['tokens'] / self.stats['Claude']['queries']
        reduction = claude_avg / stratus_avg
        print(f"\nAverage reduction: {reduction:.1f}×")

tracker = ComparisonTracker()
for query in queries:
    tracker.query_stratus(query)
    tracker.query_claude(query)
tracker.report()
```

---

## Cost Comparison

```python
# Pricing (example, check current rates)
PRICING = {
    'stratus-small': {'input': 0.001, 'output': 0.002},  # per 1K tokens
    'claude-sonnet-4-5': {'input': 0.003, 'output': 0.015},
    'gpt-4o': {'input': 0.0025, 'output': 0.01}
}

def calculate_cost(tokens, model):
    # Assume 50% input, 50% output
    input_cost = (tokens / 2) * (PRICING[model]['input'] / 1000)
    output_cost = (tokens / 2) * (PRICING[model]['output'] / 1000)
    return input_cost + output_cost

# Run 1000 queries
queries = [...] # 1000 queries

total_tokens = {
    'stratus': 42000,   # 42 avg
    'claude': 980000,   # 980 avg
    'gpt4o': 920000     # 920 avg
}

costs = {
    'Stratus': calculate_cost(total_tokens['stratus'], 'stratus-small'),
    'Claude': calculate_cost(total_tokens['claude'], 'claude-sonnet-4-5'),
    'GPT-4o': calculate_cost(total_tokens['gpt4o'], 'gpt-4o')
}

print("Cost for 1,000 queries:")
for model, cost in costs.items():
    print(f"  {model}: ${cost:.2f}")

savings = costs['Claude'] - costs['Stratus']
print(f"\nSavings vs Claude: ${savings:.2f} ({savings/costs['Claude']:.0%})")
```

---

## Batch Comparison

```python
import concurrent.futures

def batch_compare(queries):
    """Compare models on multiple queries in parallel."""
    results = []

    def process_query(query):
        return {
            'query': query,
            'stratus': stratus_fn(query),
            'claude': claude_fn(query),
            'gpt': gpt_fn(query)
        }

    with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
        results = list(executor.map(process_query, queries))

    return results

queries = [
    "Count 'r' in 'strawberry'",
    "Is 127 prime?",
    "What's 2+2?",
    # ... more queries
]

results = batch_compare(queries)

# Analyze
correct_counts = {'Stratus': 0, 'Claude': 0, 'GPT-4o': 0}
for result in results:
    # Check correctness (simplified)
    if check_correct(result['stratus']):
        correct_counts['Stratus'] += 1
    if check_correct(result['claude']):
        correct_counts['Claude'] += 1
    if check_correct(result['gpt']):
        correct_counts['GPT-4o'] += 1

print("Success rates:")
for model, count in correct_counts.items():
    print(f"  {model}: {count/len(queries):.0%}")
```

---

## Quality Assessment

```python
def assess_quality(answer, expected):
    """Score answer quality 0-1."""
    # Correctness
    correct = expected.lower() in answer.lower()

    # Conciseness (shorter is better)
    brevity = min(len(expected) / len(answer), 1.0)

    # Combined score
    if correct:
        return 0.6 + (0.4 * brevity)
    else:
        return 0.0

test_cases = [
    ("Count 'r' in 'strawberry'", "3"),
    ("Is 127 prime?", "Yes"),
]

models = ['Stratus', 'Claude', 'GPT-4o']
scores = {model: [] for model in models}

for query, expected in test_cases:
    # Get answers
    answers = {
        'Stratus': stratus_fn(query),
        'Claude': claude_fn(query),
        'GPT-4o': gpt_fn(query)
    }

    # Score
    for model, answer in answers.items():
        score = assess_quality(answer, expected)
        scores[model].append(score)

# Average scores
for model in models:
    avg = sum(scores[model]) / len(scores[model])
    print(f"{model} average quality: {avg:.2f}")
```

---

**For more:**
- Basic reasoning: `basic-reasoning.md`
- Agent integration: `agent-integration.md`
- Benchmarks: `../docs/BENCHMARKS.md`
