# Stratus X1-AC Architecture Deep Dive

Complete technical documentation of the Stratus X1-AC (Action-Conditioned) world model architecture.

## Table of Contents

1. [Overview](#overview)
2. [Core Components](#core-components)
3. [Training Methodology](#training-methodology)
4. [Model Sizes](#model-sizes)
5. [Technical Specifications](#technical-specifications)
6. [Why This Architecture Works](#why-this-architecture-works)

---

## Overview

**Stratus X1-AC is a Predictive Action Model** - not an LLM, but a world model that predicts how digital environments respond to actions.

### Key Characteristics

- **Operates in embedding space**: 768-dimensional state representations
- **Self-supervised learning**: Trained on (state, action, next_state) tuples
- **Hybrid architecture**: Combines world model + policy head
- **Based on V-JEPA 2**: Meta's validated research, adapted for text/agents
- **Production-ready**: 46,000 lines of code, comprehensive benchmarking

### What It Predicts

```
World Model: state_t + action → state_t+1
Policy Head: (current_state, goal_state) → optimal_action
```

---

## Core Components

### 1. State Encoder

**Purpose:** Converts raw observations into semantic embeddings

**Architecture:**
- Transformer-based encoder
- Output: 768-dimensional embeddings
- Learns semantic structure from inputs

**What it does:**
- Processes text observations (HTML, code, logs, etc.)
- Extracts decision-relevant features
- Creates compressed representations
- Enables comparison in embedding space

**Example:**
```
Input: "Search results for hotels in SF: 147 listings found"
Output: [768-dim vector representing search result state]
```

### 2. Target Encoder

**Purpose:** Provides teaching signal via Exponential Moving Average (EMA)

**Architecture:**
- EMA teacher copy of state encoder
- Momentum: 0.996 → 1.0 during training
- Stop-gradient prevents trivial solutions

**Why EMA is critical:**
- Creates stable teaching targets
- Prevents representation collapse
- Enables self-supervised learning
- No human labels needed

**Training dynamics:**
```python
# Simplified EMA update
target_encoder.parameters = (
    momentum * target_encoder.parameters +
    (1 - momentum) * state_encoder.parameters
)
```

### 3. Action Encoder

**Purpose:** Embeds ~100 abstract action types into vector space

**Action vocabulary:**
- **Retrieval**: search, read, extract, scan
- **Navigation**: select, click, navigate, scroll
- **Interaction**: type, submit, edit, delete
- **System**: execute, wait, verify

**Action conditioning:**
- Actions are embedded into same space as states
- Enables "state + action → next_state" prediction
- Domain-agnostic (works across web, code, CLI, etc.)

**Example:**
```
Action: "search"
Embedding: [768-dim vector]
Combined with state: [state_vec + action_vec] → predictor
```

### 4. Predictor (World Model)

**Purpose:** Predicts future state representations

**Architecture:**
- Narrower/deeper network than encoders
- Input: current_state + action
- Output: predicted next_state representation

**What it enables:**
- **Planning by simulation**: Try actions mentally before execution
- **Multi-step rollout**: Predict 5-10 steps ahead
- **Outcome validation**: Check if action moves toward goal
- **Error recovery**: Detect when actual state ≠ predicted state

**Inference:**
```python
def predict_next_state(current_state, action):
    # Encode inputs
    state_emb = state_encoder(current_state)
    action_emb = action_encoder(action)

    # Predict next state
    next_state_pred = predictor(state_emb, action_emb)

    return next_state_pred
```

### 5. Policy Head

**Purpose:** Directly predicts optimal action from (state, goal) pairs

**Architecture:**
- Takes current state + goal state
- Outputs action probabilities
- Learned from millions of successful trajectories

**What it enables:**
- Fast inference for known patterns
- Direct decision-making without rollout
- Complements world model predictions
- Best-of-both: planning + policy

**When to use:**
- **Policy head**: Fast, known patterns (99% of cases)
- **World model**: Novel situations, need to simulate

**Inference:**
```python
def predict_action(current_state, goal_state):
    # Encode states
    current_emb = state_encoder(current_state)
    goal_emb = state_encoder(goal_state)

    # Predict action
    action_logits = policy_head(current_emb, goal_emb)
    action = sample(action_logits)

    return action
```

---

## Training Methodology

### Self-Supervised Learning

**No human labels required.** The model learns from state transition tuples:

```
(state_t, action, state_t+1)
```

**Training signal:** Predict state_t+1 given state_t + action

### Two-Loss Training (V-JEPA 2 Style)

**70% Teacher Forcing:**
- Uses ground truth states at each step
- Stable learning foundation
- Learns accurate state transitions

**30% Rollout:**
- Uses model's own predictions autoregressively
- Learns multi-step planning
- Learns error recovery
- Prevents compounding errors

**Combined loss:**
```
L_total = 0.7 × L_teacher_forcing + 0.3 × L_rollout
```

**Why this works:**
- Teacher forcing: Accurate single-step predictions
- Rollout: Multi-step consistency
- Together: Best of both worlds

### Training Data Sources (1.5T+ tokens)

**Web browsing:**
- WebArena navigation traces
- Mind2Web interaction sequences
- Real user workflows

**Code execution:**
- InterCode tasks
- Jupyter notebook sessions
- Terminal command sequences

**Information seeking:**
- HotpotQA multi-hop reasoning
- Research task trajectories
- Document navigation

**Tool use:**
- API call sequences
- Function invocations
- System commands

**Multi-step agent tasks:**
- Long-horizon problem solving
- Complex workflows
- Error recovery examples

### Training Infrastructure

**Hardware:**
- 8× H100 80GB GPUs (production target)
- Mixed precision (bf16 for H100, fp16 fallback)
- Distributed training across GPUs

**Training time:**
- V1 (4-layer, 125M params): 48 hours
- V2 (12-layer, 350M params): 1 month (target)

**Checkpointing:**
- Student encoder weights
- Teacher encoder weights
- Optimizer state
- Scheduler state
- Training step counter

**Curriculum learning:**
- Start with simple tasks
- Progressively increase complexity
- Multi-step sequences last

**Continuous learning:**
- Live agent deployments feed back
- Model improves from real usage
- Adapts to new domains

---

## Model Sizes

### Debug (16M parameters)

**Architecture:**
- Hidden size: 128
- Layers: 2
- Heads: 4

**Use case:**
- Fast iteration
- Testing changes
- Debugging training

**Performance:**
- Latency: <5ms
- Throughput: 2000+ pred/s

### Small (125M parameters)

**Architecture:**
- Hidden size: 512
- Layers: 8
- Heads: 8

**Use case:**
- Development
- Baseline comparison
- Cost-optimized production

**Performance:**
- Latency: <10ms
- Throughput: 1000+ pred/s
- Token reduction: 20-30×

### Medium (350M parameters)

**Architecture:**
- Hidden size: 768
- Layers: 12
- Heads: 12

**Use case:**
- Production deployment
- Maximum accuracy
- Complex tasks

**Performance:**
- Latency: <15ms
- Throughput: 500+ pred/s
- Token reduction: 25-35×

---

## Technical Specifications

### Embedding Space

**Dimensions:** 768 (medium), 512 (small)

**What's in the embedding:**
- Semantic content
- State structure
- Decision-relevant features
- Compressed meaning

**Not in the embedding:**
- Raw text tokens
- Irrelevant details
- Formatting artifacts

### Context Window

**Stratus processes full observation history:**
- No hard token limit
- Compression in embedding space
- Maintains decision context

**vs LLM context:**
- LLMs: Limited by token count (8K, 32K, 200K)
- Stratus: Compressed in embedding space
- Result: Effectively infinite context for agent tasks

### Inference Performance

**Latency:**
- Single prediction: <10ms (small), <15ms (medium)
- Batched: 1000+ predictions/second
- Net improvement: LLM processes fewer tokens

**Memory:**
- Model: ~500MB (small), ~1.4GB (medium)
- Inference: ~1.7GB GPU memory total
- Shared across requests

**Throughput:**
- Small: 1000+ pred/s
- Medium: 500+ pred/s
- Scales linearly with batch size

---

## Why This Architecture Works

### 1. Operates in Embedding Space

**Not token prediction:**
- LLMs predict next token
- Stratus predicts next state representation
- Semantic compression vs text generation

**Benefits:**
- Captures meaning, not words
- Robust to paraphrasing
- Domain-agnostic
- Faster inference

### 2. Self-Supervised Learning

**No labels needed:**
- Learns from state transitions themselves
- Scalable to any domain
- Continuous improvement from usage

**vs supervised learning:**
- Supervised: Needs (input, label) pairs
- Stratus: Uses (state, action, next_state)
- Result: Learns cause-and-effect directly

### 3. Predictive Capability

**Enables planning by simulation:**
- Try actions mentally
- Validate before execution
- Multi-step lookahead
- Error recovery

**vs reactive agents:**
- Reactive: Execute → observe → react
- Stratus: Predict → validate → execute
- Result: Fewer mistakes, better plans

### 4. Hybrid Architecture

**World model:**
- "What happens if I do X?"
- Simulation before action
- Multi-step planning

**Policy head:**
- "What should I do?"
- Fast inference
- Learned from successful trajectories

**Together:**
- Policy for known patterns (fast)
- World model for novel situations (safe)
- Best of both approaches

### 5. Based on Proven Research

**V-JEPA 2 (Meta):**
- Validated at scale
- Published research
- Open methodology

**Adapted for agents:**
- Text/code environments
- Digital actions
- Multi-step tasks

**Production implementation:**
- 46,000 lines of code
- Comprehensive testing
- Real-world validation

---

## Comparison: Stratus vs LLMs

| Aspect | LLMs | Stratus X1-AC |
|--------|------|---------------|
| **Prediction** | Next token | Next state |
| **Space** | Token space | Embedding space |
| **Training** | Next-token prediction | State transition prediction |
| **Planning** | Chain-of-thought | Multi-step rollout |
| **Context** | Token limit (8K-200K) | Compressed (effectively unlimited) |
| **Speed** | Token-by-token | Parallel prediction |
| **Use case** | Text generation | Action prediction |

**They're complementary:**
- Stratus: Predicts actions and states
- LLM: Reasons about predictions
- Together: Better agents

---

## Future Directions

### Stratus M1 (Multimodal)

- Text + images + video
- Shared priors for robotics
- Scene understanding
- Action grounding

### Stratus M2 (Multi-Agent)

- Multi-robot coordination
- Shared environment understanding
- Autonomy orchestration
- Collaborative planning

---

## Summary

**Stratus X1-AC is a hybrid world model + policy architecture that:**

✅ Predicts state transitions in embedding space
✅ Enables multi-step planning by simulation
✅ Learns self-supervised from agent interactions
✅ Compresses tokens 20-30× while maintaining accuracy
✅ Provides both world model and policy capabilities
✅ Works across domains (web, code, CLI, tools)

**This is not an LLM enhancement - it's a fundamentally different approach to agent intelligence.**

---

**For more:**
- Main skill: `../SKILL.md`
- Benchmarks: `BENCHMARKS.md`
- Integration: `INTEGRATION.md`
- API reference: `API.md`
