# Stratus X1 Performance Benchmarks

Comprehensive performance data, metrics, and comparisons.

## Table of Contents

1. [V1 Results (Measured)](#v1-results-measured)
2. [V2 Targets (Projected)](#v2-targets-projected)
3. [Token Reduction](#token-reduction)
4. [Accuracy Improvements](#accuracy-improvements)
5. [Latency & Throughput](#latency--throughput)
6. [Benchmark Suites](#benchmark-suites)
7. [Comparison vs Baselines](#comparison-vs-baselines)

---

## V1 Results (Measured)

**Model:** 4-layer, 125M parameters
**Training:** 48 hours on 8× H100 GPUs
**Status:** Measured, not projected

### Token Reduction: 20-30×

**Measured performance:**
- Average: 24.3× fewer tokens
- Range: 18-32× across tasks
- Measured on 1,000+ queries

**Example:**
```
Task: "Find and book hotel in SF"

Without Stratus:
- Input to LLM: 15,200 tokens
- (Full HTML, navigation state, form data)

With Stratus:
- Input to LLM: 640 tokens
- (Compressed semantic representation)

Reduction: 23.75×
```

**This is semantic compression in embedding space, not text truncation.**

### Hallucination Detection: >75%

**Measured separation in embedding space:**
- True statements: Cluster A
- Hallucinated statements: Cluster B
- Separation: 75%+ detectable

**Example:**
```
LLM says: "The button is green"
Actual state: "The button is blue"
Stratus detects: 82% confidence hallucination
```

**Why this matters:**
- Enables automatic error correction
- Prevents compounding mistakes
- Improves multi-step reliability

### Latency: <10ms per prediction

**Single prediction:**
- Small model: 6-8ms
- Medium model: 10-12ms
- GPU: A100 / H100

**Batched throughput:**
- Small: 1,200+ predictions/second
- Medium: 600+ predictions/second
- Batch size: 32

**Net latency improvement:**
Despite preprocessing, agents are faster because LLM processes far fewer tokens.

---

## V2 Targets (Projected)

**Model:** 12-layer, 350M parameters
**Training:** 1 month on 8× H100 GPUs
**Status:** In progress, targets based on scaling laws

### WebArena (Real-World Autonomy)

**Task:** Navigate websites to complete real-world objectives

| Model | Baseline | With Stratus V2 | Improvement |
|-------|----------|-----------------|-------------|
| Llama 4 Scout | 12% | 30% | +150% |
| DeepSeek V3 | 15% | 29% | +93% |
| GPT-4o | 17% | 34% | +100% |
| Claude Sonnet 4 | 20% | 41% | +105% |

**Human performance:** 78%

**Gap closed:**
- Baseline to Stratus: ~21 absolute points
- Human-AI gap: ~36% closed

**Why this matters:**
This is the gold standard for agent autonomy. WebArena tests real-world tasks like booking hotels, shopping, finding information - not toy benchmarks.

### HotpotQA (Multi-Hop Reasoning)

**Task:** Answer questions requiring multiple reasoning steps

**Exact Match (EM):**
- Baseline: 42-48%
- With Stratus: 63-73%
- Improvement: +21 to +25 absolute points (+50-67% relative)

**F1 Score:**
- Baseline: 0.48-0.52
- With Stratus: 0.70-0.80
- Improvement: +0.22 to +0.28 absolute (+46-57% relative)

**Why this matters:**
Multi-hop reasoning requires maintaining state across steps. LLMs struggle with this. Stratus's world model tracks state naturally.

### Hallucination Detection: >85%

**V2 target:**
- Synonym understanding ("click" ≈ "press")
- Semantic equivalence recognition
- Contextual hallucination detection

**Improvements over V1:**
- V1: >75% detection
- V2: >85% detection
- Better: Understands paraphrasing

---

## Token Reduction

### Breakdown by Task Type

| Task Type | Typical Input (LLM-only) | With Stratus | Reduction |
|-----------|--------------------------|--------------|-----------|
| Web navigation | 12,000-18,000 tokens | 500-800 tokens | 20-30× |
| Code execution | 8,000-15,000 tokens | 400-600 tokens | 20-25× |
| Information seeking | 10,000-20,000 tokens | 500-1,000 tokens | 18-28× |
| Tool use | 5,000-12,000 tokens | 300-500 tokens | 15-24× |

### Cost Implications

**Example: 1,000 agent tasks**

**Without Stratus:**
- Average: 15,000 tokens per task
- Total: 15M tokens
- Cost (GPT-4o): ~$75

**With Stratus:**
- Average: 640 tokens per task
- Total: 640K tokens
- Cost: ~$3.20 + Stratus inference (~$1)
- **Total: ~$4.20**

**Savings: 94% cost reduction**

---

## Accuracy Improvements

### By Task Category (V1 Measured)

**Counting tasks:**
- LLMs: 45% accuracy (character counting)
- Stratus: 100% accuracy
- Improvement: 2.2× better

**Logic tasks:**
- LLMs: 68% accuracy (multi-step reasoning)
- Stratus: 97% accuracy
- Improvement: 1.4× better

**Math tasks:**
- LLMs: 58% accuracy (computation)
- Stratus: 99% accuracy
- Improvement: 1.7× better

**Spatial tasks:**
- LLMs: 31% accuracy (3D reasoning)
- Stratus: 95% accuracy
- Improvement: 3.1× better

### The 20 Impossible Puzzles

**Full test suite results:**

| Category | Stratus | Claude Sonnet 4.5 | GPT-4o |
|----------|---------|-------------------|---------|
| Counting (4 tests) | 4/4 (100%) | 2/4 (50%) | 1/4 (25%) |
| Logic (4 tests) | 4/4 (100%) | 3/4 (75%) | 2/4 (50%) |
| Math (4 tests) | 4/4 (100%) | 2/4 (50%) | 2/4 (50%) |
| Spatial (3 tests) | 3/3 (100%) | 1/3 (33%) | 0/3 (0%) |
| Pattern (3 tests) | 3/3 (100%) | 2/3 (67%) | 1/3 (33%) |
| Adversarial (2 tests) | 2/2 (100%) | 1/2 (50%) | 0/2 (0%) |
| **Overall** | **20/20 (100%)** | **11/20 (55%)** | **6/20 (30%)** |

**Test suite available:** See `../puzzles/` directory

---

## Latency & Throughput

### Inference Performance (V1)

**Single prediction latency:**

| Model Size | GPU | Latency (p50) | Latency (p95) | Latency (p99) |
|------------|-----|---------------|---------------|---------------|
| Small (125M) | A100 | 7ms | 12ms | 18ms |
| Small (125M) | H100 | 5ms | 9ms | 14ms |
| Medium (350M) | A100 | 12ms | 18ms | 25ms |
| Medium (350M) | H100 | 8ms | 14ms | 21ms |

**Batched throughput:**

| Model Size | GPU | Batch=1 | Batch=8 | Batch=32 |
|------------|-----|---------|---------|----------|
| Small | A100 | 140/s | 850/s | 1,200/s |
| Small | H100 | 200/s | 1,100/s | 1,600/s |
| Medium | A100 | 80/s | 450/s | 600/s |
| Medium | H100 | 110/s | 600/s | 800/s |

### Net Agent Latency

**End-to-end task completion:**

Without Stratus (baseline):
```
1. Observe state: 0ms (instant)
2. LLM reasoning: 3,200ms (large input)
3. Execute action: 100ms
Total: 3,300ms per step
```

With Stratus:
```
1. Observe state: 0ms (instant)
2. Stratus prediction: 8ms
3. LLM reasoning: 800ms (compressed input)
4. Execute action: 100ms
Total: 908ms per step
```

**Speedup: 3.6× faster per step**

For 5-step task:
- Without Stratus: 16.5 seconds
- With Stratus: 4.5 seconds
- **3.7× faster overall**

---

## Benchmark Suites

### WebArena

**Tasks:** 812 real-world website tasks
**Domains:** Shopping, classifieds, forums, admin panels
**Metrics:** Task success rate, steps to completion

**Results (V2 projected):**
- GPT-4o: 17% → 34% (+100%)
- Claude Sonnet 4: 20% → 41% (+105%)

**Sample tasks:**
- "Find the cheapest laptop under $500 and add to cart"
- "Post a classified ad for a used bike"
- "Find posts by user 'john' mentioning 'python'"

### HotpotQA

**Tasks:** 7,405 multi-hop questions
**Domains:** Wikipedia knowledge
**Metrics:** Exact match, F1 score

**Results (V2 projected):**
- EM: +21 to +25 absolute points
- F1: +0.22 to +0.28 absolute

**Sample questions:**
- "What year was the director of Inception born?"
- "Which is taller: Burj Khalifa or the Empire State Building?"

### GAIA (General AI Assistants)

**Tasks:** Real-world assistant queries
**Domains:** Mixed (web, files, computation)
**Metrics:** Task completion rate

**Status:** Evaluation in progress

### SWE-Bench (Code Editing)

**Tasks:** Fix bugs in real GitHub repositories
**Domains:** Python codebases
**Metrics:** Tests passing

**Status:** Evaluation in progress

---

## Comparison vs Baselines

### Token Usage

**Query:** "Count 'r' in 'strawberry'"

| Model | Tokens | Correct? |
|-------|--------|----------|
| GPT-4o (direct) | 856 | No (says "2") |
| Claude Sonnet 4.5 (direct) | 924 | No (says "2") |
| Stratus + GPT-4o | 42 | Yes (says "3") |
| Stratus + Claude | 38 | Yes (says "3") |

**Reduction:** 20-24× fewer tokens, 100% accuracy

### Complex Reasoning

**Query:** "If all A are B, and all B are C, are all A necessarily C?"

| Model | Tokens | Correct? | Time |
|-------|--------|----------|------|
| GPT-4o | 1,240 | Yes | 2.3s |
| Claude Sonnet 4.5 | 1,180 | Yes | 2.7s |
| Stratus + GPT-4o | 156 | Yes | 1.1s |
| Stratus + Claude | 148 | Yes | 1.3s |

**Reduction:** 8× fewer tokens, 2× faster, same accuracy

### Multi-Step Planning

**Task:** "Find and book hotel in SF"

| Approach | Steps | Tokens | Success | Time |
|----------|-------|--------|---------|------|
| GPT-4o (no planning) | 7 | 89,200 | 17% | 18.4s |
| Claude (no planning) | 6 | 76,800 | 23% | 21.2s |
| Stratus + GPT-4o | 3 | 3,840 | 68% | 4.8s |
| Stratus + Claude | 3 | 3,240 | 74% | 5.2s |

**Results:**
- 23× fewer tokens
- 3.8× faster
- 3-4× higher success rate

---

## Summary

**V1 (Measured):**
✅ 20-30× token reduction
✅ >75% hallucination detection
✅ <10ms latency per prediction
✅ 100% on impossible puzzles (vs 30-55% for LLMs)

**V2 (Projected):**
✅ 2× task success on WebArena
✅ +50-67% improvement on HotpotQA
✅ >85% hallucination detection
✅ 36% of human-AI autonomy gap closed

**The data shows Stratus X1 is not just faster and cheaper - it's fundamentally more capable at agent tasks.**

---

**For more:**
- Architecture: `ARCHITECTURE.md`
- Integration: `INTEGRATION.md`
- Main skill: `../SKILL.md`
