# Agent Integration Examples

Complete examples of integrating Stratus into agent architectures.

## Simple Agent Loop

```python
from openai import OpenAI
import requests

class StratusAgent:
    def __init__(self, api_key):
        self.client = OpenAI(
            base_url="http://212.115.124.137:8000/v1",
            api_key=api_key
        )
        self.model = "stratus-x1ac-small-claude-sonnet-4-5"

    def plan(self, goal, max_steps=5):
        """Use rollout endpoint to plan actions."""
        response = requests.post(
            "http://212.115.124.137:8000/v1/rollout",
            headers={"Authorization": f"Bearer {self.client.api_key}"},
            json={
                "goal": goal,
                "max_steps": max_steps,
                "return_intermediate": True
            }
        )
        return response.json()

    def execute(self, action, state):
        """Execute action using chat completions."""
        response = self.client.chat.completions.create(
            model=self.model,
            messages=[{
                "role": "user",
                "content": f"Execute '{action}' given state: {state}"
            }]
        )
        return response.choices[0].message.content

    def run(self, goal):
        """Full agent loop: plan → execute → validate."""
        # Plan
        plan = self.plan(goal)
        actions = plan['summary']['action_path']

        print(f"Goal: {goal}")
        print(f"Plan: {' → '.join(actions)}")

        # Execute
        state = "initial"
        for i, action in enumerate(actions):
            print(f"\nStep {i+1}: {action}")
            result = self.execute(action, state)
            state = result
            print(f"Result: {result[:100]}...")

        # Validate
        print(f"\nOutcome: {plan['summary']['outcome']}")
        return state

# Use the agent
agent = StratusAgent("stratus_sk_live_...")
agent.run("Find and book hotel in SF")
```

---

## ReAct-Style Agent

```python
class ReActAgent:
    def __init__(self, api_key, tools):
        self.client = OpenAI(
            base_url="http://212.115.124.137:8000/v1",
            api_key=api_key
        )
        self.model = "stratus-x1ac-medium-claude-sonnet-4-5"
        self.tools = tools

    def think(self, observation):
        """Generate next thought/action."""
        response = self.client.chat.completions.create(
            model=self.model,
            messages=[{
                "role": "user",
                "content": f"Observation: {observation}\n\nWhat should I do next?"
            }]
        )
        return response.choices[0].message.content

    def act(self, action, args):
        """Execute tool."""
        if action in self.tools:
            return self.tools[action](*args)
        raise ValueError(f"Unknown action: {action}")

    def run(self, task, max_steps=10):
        """ReAct loop."""
        observation = f"Task: {task}"

        for step in range(max_steps):
            # Think
            thought_action = self.think(observation)
            print(f"Step {step+1}: {thought_action}")

            # Parse action
            if "FINISH" in thought_action:
                return thought_action

            action, *args = thought_action.split()

            # Act
            observation = self.act(action, args)
            print(f"Observation: {observation}")

        return "Max steps reached"

# Define tools
tools = {
    "search": lambda q: f"Found {q} results",
    "read": lambda url: f"Content of {url}",
    "execute": lambda cmd: f"Executed {cmd}"
}

agent = ReActAgent("stratus_sk_live_...", tools)
agent.run("Research competitor pricing")
```

---

## LangChain Integration

```python
from langchain.llms import OpenAI as LangChainOpenAI
from langchain.agents import initialize_agent, Tool
from langchain.agents import AgentType

# Configure Stratus as LLM
llm = LangChainOpenAI(
    openai_api_base="http://212.115.124.137:8000/v1",
    openai_api_key="stratus_sk_live_...",
    model_name="stratus-x1ac-small-gpt-4o"
)

# Define tools
tools = [
    Tool(
        name="Search",
        func=lambda q: f"Search results for {q}",
        description="Search for information"
    ),
    Tool(
        name="Calculate",
        func=lambda expr: eval(expr),
        description="Calculate mathematical expressions"
    )
]

# Initialize agent
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

# Run
result = agent.run("What is 17 squared plus 25?")
print(result)
```

---

## AutoGen Integration

```python
from autogen import AssistantAgent, UserProxyAgent

# Stratus-powered assistant
assistant = AssistantAgent(
    name="stratus_assistant",
    llm_config={
        "config_list": [{
            "model": "stratus-x1ac-medium-claude-sonnet-4-5",
            "base_url": "http://212.115.124.137:8000/v1",
            "api_key": "stratus_sk_live_..."
        }],
        "temperature": 0.7
    }
)

# User proxy for execution
user_proxy = UserProxyAgent(
    name="user",
    human_input_mode="NEVER",
    code_execution_config={"work_dir": "coding"}
)

# Start conversation
user_proxy.initiate_chat(
    assistant,
    message="Write a Python function to check if a number is prime."
)
```

---

## Error Recovery Agent

```python
class RobustAgent:
    def __init__(self, api_key):
        self.stratus = OpenAI(
            base_url="http://212.115.124.137:8000/v1",
            api_key=api_key
        )
        self.fallback = OpenAI(api_key="sk-...")  # Direct LLM

    def execute_with_recovery(self, goal):
        """Execute with automatic error recovery."""
        max_retries = 3
        attempt = 0

        while attempt < max_retries:
            try:
                # Plan with Stratus
                plan_response = requests.post(
                    "http://212.115.124.137:8000/v1/rollout",
                    headers={"Authorization": f"Bearer {self.stratus.api_key}"},
                    json={"goal": goal, "max_steps": 5, "return_intermediate": True}
                )
                plan = plan_response.json()

                # Execute plan
                for action in plan['summary']['action_path']:
                    result = self.execute_action(action)

                    # Validate with Stratus
                    if not self.validate_result(result, action):
                        print(f"Error detected, recovering...")
                        raise ValueError("Invalid result")

                return result

            except Exception as e:
                print(f"Attempt {attempt + 1} failed: {e}")
                attempt += 1

                if attempt == max_retries:
                    # Fallback to direct LLM
                    print("Using fallback LLM...")
                    return self.fallback_execution(goal)

    def validate_result(self, result, expected_action):
        """Check if result matches expected outcome."""
        # Compare actual vs predicted state
        # Return True if they match
        return True  # Simplified

    def fallback_execution(self, goal):
        """Execute with direct LLM (no Stratus)."""
        response = self.fallback.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": goal}]
        )
        return response.choices[0].message.content

agent = RobustAgent("stratus_sk_live_...")
result = agent.execute_with_recovery("Find hotel in SF")
```

---

## Monitoring Agent

```python
class MonitoredAgent:
    def __init__(self, api_key):
        self.client = OpenAI(
            base_url="http://212.115.124.137:8000/v1",
            api_key=api_key
        )
        self.metrics = {
            "tokens_saved": 0,
            "queries": 0,
            "successes": 0
        }

    def query(self, prompt):
        """Query with metrics tracking."""
        response = self.client.chat.completions.create(
            model="stratus-x1ac-small-gpt-4o",
            messages=[{"role": "user", "content": prompt}]
        )

        # Track metrics
        self.metrics["queries"] += 1
        tokens = response.usage.total_tokens

        stratus_meta = response.model_extra.get('stratus', {})
        reduction = stratus_meta.get('token_reduction', 1)

        # Estimate tokens without Stratus
        estimated_without = tokens * reduction
        self.metrics["tokens_saved"] += (estimated_without - tokens)

        return response.choices[0].message.content

    def report(self):
        """Show performance metrics."""
        print(f"Queries: {self.metrics['queries']}")
        print(f"Tokens saved: {self.metrics['tokens_saved']:,}")
        print(f"Avg reduction: {self.metrics['tokens_saved'] / self.metrics['queries']:.1f}× per query")

agent = MonitoredAgent("stratus_sk_live_...")
for query in queries:
    agent.query(query)
agent.report()
```

---

**For more:**
- Basic reasoning: `basic-reasoning.md`
- Rollout planning: `rollout-planning.md`
- Integration guide: `../docs/INTEGRATION.md`
