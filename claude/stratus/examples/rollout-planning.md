# Rollout Planning Examples

Using the `/v1/rollout` endpoint for multi-step action prediction and validation.

## Goal-Based Planning

**Let Stratus predict the action sequence:**

```python
import requests

API_URL = "http://212.115.124.137:8000/v1/rollout"
API_KEY = "stratus_sk_live_..."

response = requests.post(
    API_URL,
    headers={"Authorization": f"Bearer {API_KEY}"},
    json={
        "goal": "Find and book hotel in San Francisco",
        "max_steps": 5,
        "return_intermediate": True
    }
)

plan = response.json()

# Get predicted action sequence
actions = plan['summary']['action_path']
print("Actions:", " → ".join(actions))
# Output: "search → select → read → type → submit"

# Check outcome
outcome = plan['summary']['outcome']
print("Outcome:", outcome)
# Output: "Goal likely achieved (large cumulative change)"

# Get total state change
change = plan['summary']['total_state_change']
print(f"State change: {change:.2f}")
# Output: "State change: 28.45"
```

---

## Validating Action Sequences

**Test your planned approach:**

```python
def validate_plan(goal, actions):
    """
    Test if an action sequence will achieve the goal.

    Args:
        goal: Task description
        actions: List of action IDs or names

    Returns:
        bool: True if plan likely succeeds
    """
    response = requests.post(
        "http://212.115.124.137:8000/v1/rollout",
        headers={"Authorization": f"Bearer {API_KEY}"},
        json={
            "goal": goal,
            "actions": actions,
            "return_intermediate": True
        }
    )

    result = response.json()
    outcome = result['summary']['outcome']

    return "Goal likely achieved" in outcome

# Test a plan
goal = "Debug failing test case"
plan_a = [17, 28, 42, 50]  # search, read, edit, execute
plan_b = [28, 42, 50]      # read, edit, execute (skip search)

if validate_plan(goal, plan_a):
    print("Plan A: Valid")
else:
    print("Plan A: Invalid, trying Plan B")
    if validate_plan(goal, plan_b):
        print("Plan B: Valid")
```

---

## Multi-Step Simulation

**Inspect each step:**

```python
response = requests.post(..., json={
    "goal": "Research competitor pricing",
    "max_steps": 3,
    "return_intermediate": True
})

plan = response.json()

# Analyze each step
for pred in plan['predictions']:
    step = pred['step']
    action = pred['action']['action_name']
    state_change = pred['state_change']
    interpretation = pred['interpretation']

    print(f"\nStep {step}: {action}")
    print(f"  State change: {state_change:.2f}")
    print(f"  Interpretation: {interpretation}")
```

**Output:**
```
Step 1: search
  State change: 7.80
  Interpretation: Query narrows search space

Step 2: select
  State change: 8.30
  Interpretation: Selection focuses on specific option

Step 3: read
  State change: 7.30
  Interpretation: Information retrieved
```

---

## Comparing Multiple Paths

**Find the best approach:**

```python
def score_path(goal, actions):
    """Score an action path by total state change."""
    response = requests.post(
        "http://212.115.124.137:8000/v1/rollout",
        headers={"Authorization": f"Bearer {API_KEY}"},
        json={
            "goal": goal,
            "actions": actions,
            "return_intermediate": True
        }
    )

    result = response.json()
    return result['summary']['total_state_change']

goal = "Find hotel in San Francisco"

# Compare approaches
path1 = [17, 10, 28]                # search, select, read
path2 = [17, 35, 10, 28]            # search, filter, select, read
path3 = [17, 10, 35, 28]            # search, select, filter, read

paths = {
    "Direct": path1,
    "Filter first": path2,
    "Filter after select": path3
}

scores = {name: score_path(goal, actions) for name, actions in paths.items()}

best = max(scores, key=scores.get)
print(f"Best path: {best} (score: {scores[best]:.2f})")
# Output: "Best path: Filter first (score: 32.45)"
```

---

## Planning with Context

**Include initial state:**

```python
response = requests.post(..., json={
    "goal": "Complete checkout",
    "max_steps": 4,
    "initial_state": "User has items in cart, on product page",
    "return_intermediate": True
})

plan = response.json()
# Stratus takes initial state into account when planning
```

---

## Detecting Bad Plans

**Check for low state change:**

```python
def plan_and_validate(goal, max_steps=5):
    response = requests.post(..., json={
        "goal": goal,
        "max_steps": max_steps,
        "return_intermediate": True
    })

    plan = response.json()
    change = plan['summary']['total_state_change']

    if change < 10:
        print(f"Warning: Low state change ({change:.2f})")
        print("Plan may not achieve goal")
        return None

    if change < 20:
        print(f"Moderate progress expected ({change:.2f})")
    else:
        print(f"Goal likely achieved ({change:.2f})")

    return plan

plan = plan_and_validate("Find product reviews")
```

---

## Adaptive Planning

**Replan if needed:**

```python
def execute_with_replanning(goal):
    # Initial plan
    plan = requests.post(..., json={
        "goal": goal,
        "max_steps": 5,
        "return_intermediate": True
    }).json()

    actions = plan['summary']['action_path']

    for i, action in enumerate(actions):
        # Execute action
        actual_result = execute_action(action)

        # Validate against prediction
        predicted_state = plan['predictions'][i]['predicted_state']

        if not states_match(actual_result, predicted_state):
            print(f"State mismatch at step {i}, replanning...")

            # Replan from current state
            new_plan = requests.post(..., json={
                "goal": goal,
                "max_steps": len(actions) - i,
                "initial_state": actual_result,
                "return_intermediate": True
            }).json()

            # Continue with new plan
            actions = actions[:i] + new_plan['summary']['action_path']

    return "Goal achieved"
```

---

## Batch Planning

**Plan multiple tasks:**

```python
import concurrent.futures

goals = [
    "Find hotel in SF",
    "Book flight to NYC",
    "Research competitor pricing"
]

def plan_task(goal):
    response = requests.post(..., json={
        "goal": goal,
        "max_steps": 5,
        "return_intermediate": True
    })
    return response.json()

# Plan in parallel
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    plans = list(executor.map(plan_task, goals))

for goal, plan in zip(goals, plans):
    print(f"{goal}: {plan['summary']['action_path']}")
```

---

## Interpreting State Magnitudes

```python
def interpret_magnitude(mag):
    if mag < 10:
        return "Low complexity"
    elif mag < 15:
        return "Medium complexity"
    elif mag < 20:
        return "High complexity"
    else:
        return "Very high complexity"

plan = response.json()
initial = plan['summary']['initial_magnitude']
final = plan['summary']['final_magnitude']

print(f"Initial: {interpret_magnitude(initial)}")
print(f"Final: {interpret_magnitude(final)}")
```

---

**For more:**
- Basic reasoning: `basic-reasoning.md`
- Agent integration: `agent-integration.md`
- API reference: `../docs/API.md`
