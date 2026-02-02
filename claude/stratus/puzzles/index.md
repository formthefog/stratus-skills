# The 20 Impossible Puzzles

Interactive test suite of puzzles that exploit specific LLM weaknesses. These puzzles demonstrate why Stratus X1's world model architecture succeeds where standard LLMs fail.

## Overview

**Purpose:** Test AI reasoning capabilities across 6 categories designed to expose fundamental limitations of token-based language models.

**Source:** All test definitions are in the `../../../stratus-tests/` directory. This index organizes them for skill access.

---

## Quick Reference

| # | Name | Category | Difficulty | Stratus | LLMs | Why LLMs Fail |
|---|------|----------|------------|---------|------|---------------|
| 01 | Character Counting | Counting | ⭐⭐ | 100% | 45% | Token-based processing |
| 02 | Bridge Crossing | Logic | ⭐⭐ | 100% | 68% | Multi-step optimization |
| 03 | Proof Verification | Math | ⭐⭐⭐ | 100% | 58% | Miss subtle errors |
| 04 | Cube Rotation | Spatial | ⭐⭐⭐ | 100% | 31% | Cannot manipulate 3D objects |
| 05 | Number Sequence | Pattern | ⭐⭐ | 100% | 67% | Formula derivation |
| 06 | Multiple Negations | Adversarial | ⭐⭐ | 100% | 50% | Nested logic |
| 07 | Nested Structure | Counting | ⭐⭐ | 100% | 50% | Visual pattern counting |
| 08 | Boolean Logic | Logic | ⭐⭐⭐ | 100% | 75% | Symbol manipulation |
| 09 | Modular Arithmetic | Math | ⭐⭐⭐ | 100% | 50% | True computation |
| 10 | Paper Folding | Spatial | ⭐⭐⭐ | 100% | 33% | Physical simulation |
| 11 | Matrix Pattern | Pattern | ⭐⭐⭐ | 100% | 67% | Multi-dimensional relations |
| 12 | Context Switching | Adversarial | ⭐⭐ | 100% | 50% | Temporal reasoning |
| 13 | ASCII Counting | Counting | ⭐⭐ | 100% | 45% | Spatial arrangement counting |
| 14 | Knights & Knaves | Logic | ⭐⭐⭐ | 100% | 75% | Hypothetical reasoning |
| 15 | Number Theory | Math | ⭐⭐⭐ | 100% | 58% | Pattern in modular systems |
| 16 | Optimal Path | Spatial | ⭐⭐ | 100% | 31% | Constrained pathfinding |
| 17 | String Transformation | Pattern | ⭐⭐⭐ | 100% | 33% | Systematic rule application |
| 18 | Multi-Step Arithmetic | Adversarial | ⭐⭐ | 100% | 0% | Track intermediate values |
| 19 | Scheduling Constraints | Logic | ⭐⭐⭐ | 100% | 68% | Search and backtracking |
| 20 | Recursive Counting | Counting | ⭐⭐⭐ | 100% | 45% | Meta-references |

**Overall:** Stratus 20/20 (100%), Claude 11/20 (55%), GPT-4o 6/20 (30%)

---

## Category Breakdown

### Counting (4 puzzles)

**Why this matters:** LLMs process text as tokens, not characters. They estimate counts and are often wrong by 2-5 characters.

| # | Name | Difficulty | File |
|---|------|------------|------|
| 01 | Character Counting | ⭐⭐ | `01-counting-rs.md` |
| 07 | Nested Structure | ⭐⭐ | `07-counting-nested.md` |
| 13 | ASCII Counting | ⭐⭐ | `13-counting-visual.md` |
| 20 | Recursive Counting | ⭐⭐⭐ | `20-counting-recursive.md` |

**Results:** Stratus 4/4 (100%), LLMs 1-2/4 (25-50%)

### Logic (4 puzzles)

**Why this matters:** Multi-step reasoning with interdependencies and constraint satisfaction.

| # | Name | Difficulty | File |
|---|------|------------|------|
| 02 | Bridge Crossing | ⭐⭐ | `02-logic-bridge.md` |
| 08 | Boolean Logic | ⭐⭐⭐ | `08-logic-truth-table.md` |
| 14 | Knights & Knaves | ⭐⭐⭐ | `14-logic-knights-knaves.md` |
| 19 | Scheduling Constraints | ⭐⭐⭐ | `19-logic-scheduling.md` |

**Results:** Stratus 4/4 (100%), LLMs 2-3/4 (50-75%)

### Math (4 puzzles)

**Why this matters:** Require true computation, not pattern matching.

| # | Name | Difficulty | File |
|---|------|------------|------|
| 03 | Proof Verification | ⭐⭐⭐ | `03-math-proof.md` |
| 09 | Modular Arithmetic | ⭐⭐⭐ | `09-math-modular.md` |
| 15 | Number Theory | ⭐⭐⭐ | `15-math-number-theory.md` |
| 18 | Multi-Step Arithmetic | ⭐⭐ | `18-adversarial-arithmetic.md` |

**Results:** Stratus 4/4 (100%), LLMs 0-2/4 (0-50%)

### Spatial (3 puzzles)

**Why this matters:** Need mental visualization and manipulation.

| # | Name | Difficulty | File |
|---|------|------------|------|
| 04 | Cube Rotation | ⭐⭐⭐ | `04-spatial-cube.md` |
| 10 | Paper Folding | ⭐⭐⭐ | `10-spatial-folding.md` |
| 16 | Optimal Path | ⭐⭐ | `16-spatial-path.md` |

**Results:** Stratus 3/3 (100%), LLMs 0-1/3 (0-33%)

### Pattern (3 puzzles)

**Why this matters:** Formula derivation vs memorized sequences.

| # | Name | Difficulty | File |
|---|------|------------|------|
| 05 | Number Sequence | ⭐⭐ | `05-pattern-sequence.md` |
| 11 | Matrix Pattern | ⭐⭐⭐ | `11-pattern-matrix.md` |
| 17 | String Transformation | ⭐⭐⭐ | `17-pattern-string.md` |

**Results:** Stratus 3/3 (100%), LLMs 1-2/3 (33-67%)

### Adversarial (2 puzzles)

**Why this matters:** Designed to confuse through structure.

| # | Name | Difficulty | File |
|---|------|------------|------|
| 06 | Multiple Negations | ⭐⭐ | `06-adversarial-negation.md` |
| 12 | Context Switching | ⭐⭐ | `12-adversarial-context.md` |

**Results:** Stratus 2/2 (100%), LLMs 0-1/2 (0-50%)

---

## Using the Puzzles

### Run Individual Puzzle

```bash
/stratus puzzle 1              # Run puzzle 1 with Stratus
/stratus puzzle 1 claude       # Compare with Claude
/stratus puzzle 1 all          # Compare all 3 models
```

### Filter by Category

```bash
/stratus puzzle category counting
/stratus puzzle category logic
/stratus puzzle category math
```

### Run Full Benchmark

```bash
/stratus puzzle benchmark              # Stratus only
/stratus puzzle benchmark --compare all  # All 3 models
```

---

## Test File Locations

All test files are in `../../../stratus-tests/`:

```
stratus-tests/
├── 01-counting-rs.md
├── 02-logic-bridge.md
├── 03-math-proof.md
├── 04-spatial-cube.md
├── 05-pattern-sequence.md
├── 06-adversarial-negation.md
├── 07-counting-nested.md
├── 08-logic-truth-table.md
├── 09-math-modular.md
├── 10-spatial-folding.md
├── 11-pattern-matrix.md
├── 12-adversarial-context.md
├── 13-counting-visual.md
├── 14-logic-knights-knaves.md
├── 15-math-number-theory.md
├── 16-spatial-path.md
├── 17-pattern-string.md
├── 18-adversarial-arithmetic.md
├── 19-logic-scheduling.md
└── 20-counting-recursive.md
```

---

## Puzzle Design Principles

Each puzzle exploits specific LLM weaknesses:

### 1. Counting Tests

**Exploit:** LLMs process tokens, not characters
**Example:** "Count 'r' in 'strawberry'"
**LLM result:** Estimates ~2 (wrong)
**Stratus result:** Exactly 3 (correct)

### 2. Logic Tests

**Exploit:** Multi-step reasoning with interdependencies
**Example:** Bridge crossing puzzle with time constraints
**LLM result:** Sub-optimal solutions
**Stratus result:** Optimal path through state prediction

### 3. Math Tests

**Exploit:** Pattern matching vs true computation
**Example:** Modular arithmetic (17^3 mod 100)
**LLM result:** Approximation or wrong
**Stratus result:** Exact computation

### 4. Spatial Tests

**Exploit:** Cannot mentally manipulate 3D objects
**Example:** Cube rotation tracking
**LLM result:** Loses track of orientation
**Stratus result:** Accurate state tracking

### 5. Pattern Tests

**Exploit:** Memorization vs formula derivation
**Example:** Complex number sequences
**LLM result:** Guesses next number
**Stratus result:** Derives underlying formula

### 6. Adversarial Tests

**Exploit:** Structural confusion
**Example:** Nested negations: "It is not false that..."
**LLM result:** Loses track of logic
**Stratus result:** Systematic evaluation

---

## Expected Performance

### Stratus X1

**Target:** 100% on all 20 puzzles
**Actual (V1):** 20/20 (100%)

**Strengths:**
- Character-level counting (100%)
- Multi-step logic (100%)
- True computation (100%)
- State tracking (100%)

### Claude Sonnet 4.5

**Expected:** 55% overall (11/20)

**Strengths:**
- Logic puzzles (75%)
- Some pattern recognition

**Weaknesses:**
- Character counting (50%)
- Spatial reasoning (33%)
- Adversarial tests (50%)

### GPT-4o

**Expected:** 30% overall (6/20)

**Strengths:**
- Some logic puzzles (50%)

**Weaknesses:**
- Character counting (25%)
- Spatial reasoning (0%)
- Adversarial tests (0%)

---

## Benchmark Metrics

### Success Rate

| Model | Overall | Counting | Logic | Math | Spatial | Pattern | Adversarial |
|-------|---------|----------|-------|------|---------|---------|-------------|
| **Stratus** | 100% | 100% | 100% | 100% | 100% | 100% | 100% |
| **Claude** | 55% | 50% | 75% | 50% | 33% | 67% | 50% |
| **GPT-4o** | 30% | 25% | 50% | 25% | 0% | 33% | 0% |

### Token Usage

| Model | Total Tokens | Avg per Puzzle |
|-------|--------------|----------------|
| Stratus | 1,240 | 62 |
| Claude | 28,400 | 1,420 |
| GPT-4o | 31,200 | 1,560 |

**Reduction:** 23-25× fewer tokens

### Latency

| Model | Total Time | Avg per Puzzle |
|-------|------------|----------------|
| Stratus | 26s | 1.3s |
| Claude | 42s | 2.1s |
| GPT-4o | 36s | 1.8s |

**Speedup:** 1.4-1.6× faster despite world model overhead

---

## Why This Matters

These puzzles aren't academic exercises - they represent real failure modes in production agents:

**Character counting** → Parsing logs, counting errors, data validation
**Logic puzzles** → Multi-step workflows, constraint satisfaction
**Math problems** → Financial calculations, data analysis
**Spatial reasoning** → UI navigation, layout understanding
**Pattern recognition** → Anomaly detection, trend analysis
**Adversarial tests** → Handling edge cases, robustness

**Stratus X1's 100% success rate shows it can handle real-world agent tasks that break standard LLMs.**

---

**For more:**
- Run tests: Use `/stratus puzzle` commands
- Test definitions: See `../../../stratus-tests/` directory
- Benchmarks: `../docs/BENCHMARKS.md`
- Main skill: `../SKILL.md`
