# Stratus Skills

Stratus X1 is a **Predictive Action Model** that turns LLMs into agents that can **simulate, validate, and plan** before acting. These skills let any LLM tap into a world model that predicts how digital environments respond to actions — unlocking reliability, precision, and long-horizon control that plain prompting can’t reach.

If you’ve ever watched an LLM:
- Miscount characters
- Make a brittle multi-step plan
- Hallucinate a tool response
- Get lost in long-horizon tasks

Stratus X1 is the missing system. It doesn’t guess. It **predicts outcomes**.

---

## What Stratus X1 Unlocks

### 1) Reliable Action Planning
Stratus simulates action sequences before execution, so agents can **choose the path that actually works** — not the one that sounds good.

### 2) True Computation (Not Token Guessing)
Counting, arithmetic, logic verification, and structured reasoning are handled **correctly**, because Stratus operates on **state transitions**, not token heuristics.

### 3) Long-Horizon Control
Stratus predicts **how each step changes the world state**, allowing agents to maintain direction across long, multi-step tasks.

### 4) Lower Cost, Higher Accuracy
Stratus compresses meaning into a world-model embedding, achieving **20–30× token reduction** while improving task success rates.

### 5) Built for Agents
Stratus is a **world model** + **policy head**. That means agents can:
- Predict what will happen if they act
- Validate a plan before executing
- Compare multiple strategies
- Recover from failures with state-aware reasoning

---

## In One Sentence

**Stratus X1 lets your LLM plan and act like an agent that can see the future.**

## Install (Claude Code)

**When a user asks to install Stratus skills, follow these steps:**

### 1. Identify the Claude Code Skills Directory

```bash
# Check if user has .claude/skills/ directory
ls ~/.claude/skills/ 2>/dev/null || ls ./.claude/skills/ 2>/dev/null
```

**Typical locations:**
- Global: `~/.claude/skills/`
- Project-local: `./.claude/skills/`

### 2. Clone or Copy the Stratus Skill

**Option A: Clone into skills directory (recommended)**

```bash
cd ~/.claude/skills/
git clone https://github.com/formthefog/stratus-skills.git temp-stratus
cp -r temp-stratus/claude/stratus ./stratus
rm -rf temp-stratus
```

**Option B: Direct copy if repo is already cloned**

```bash
# If user has already cloned this repo somewhere
cp -r /path/to/stratus-skills/claude/stratus ~/.claude/skills/stratus
```

### 3. Verify Installation

```bash
# Check that skill exists
ls ~/.claude/skills/stratus/SKILL.md

# Should show:
# SKILL.md
# docs/
# puzzles/
# examples/
```

### 4. Test the Skill

In Claude Code, run:
```
/stratus
```

Should show the comprehensive Stratus overview and quick menu.

### 5. Environment Setup (Required)

The skill needs these environment variables:

```bash
# Add to ~/.bashrc or ~/.zshrc or .env
export STRATUS_API_KEY=stratus_sk_live_...
export STRATUS_API_URL=http://212.115.124.137:8000

# Optional (for model comparison)
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
```

---

## Install (Codex)

**When a user asks to install the Codex skill, follow these steps:**

### 1. Identify the Codex Skills Directory

```bash
# Check if user has .codex/skills/ directory
ls ~/.codex/skills/ 2>/dev/null || ls ./.codex/skills/ 2>/dev/null
```

**Typical locations:**
- Global: `~/.codex/skills/`
- Project-local: `./.codex/skills/`

### 2. Clone or Copy the Stratus Skill

**Option A: Clone into skills directory (recommended)**

```bash
cd ~/.codex/skills/
git clone https://github.com/formthefog/stratus-skills.git temp-stratus
cp -r temp-stratus/codex/stratus ./stratus
rm -rf temp-stratus
```

**Option B: Direct copy if repo is already cloned**

```bash
# If user has already cloned this repo somewhere
cp -r /path/to/stratus-skills/codex/stratus ~/.codex/skills/stratus
```

### 3. Verify Installation

```bash
# Check that skill exists
ls ~/.codex/skills/stratus/SKILL.md

# Should show:
# SKILL.md
# docs/
# puzzles/
# examples/
```

### 4. Test the Skill

In Codex, run:
```
stratus help
```

Should show the comprehensive Stratus overview and quick menu.

### 5. Environment Setup (Required)

The skill needs these environment variables:

```bash
# Add to ~/.bashrc or ~/.zshrc or .env
export STRATUS_API_KEY=stratus_sk_live_...
export STRATUS_API_URL=http://212.115.124.137:8000

# Optional (for model comparison)
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
```

---

---

## What You Get

### Comprehensive Stratus Skill (7,000+ lines)

**Main skill:** `claude/stratus/SKILL.md` (1,723 lines)
- 50+ commands across 10 categories
- Complete navigation and command reference
- Interactive 20 impossible puzzles
- Model comparison framework
- Embedded examples and guides

**Supporting docs:** `claude/stratus/docs/` (5 files)
- `ARCHITECTURE.md` - X1-AC technical deep dive
- `BENCHMARKS.md` - Performance data and metrics
- `INTEGRATION.md` - Integration patterns and guides
- `API.md` - Complete API reference
- `TROUBLESHOOTING.md` - Debug and fix issues

**Puzzle suite:** `claude/stratus/puzzles/`
- `index.md` - All 20 impossible puzzles organized

**Code examples:** `claude/stratus/examples/` (4 files)
- `basic-reasoning.md` - Counting, math, logic examples
- `rollout-planning.md` - Multi-step planning patterns
- `agent-integration.md` - Agent framework integration
- `comparison.md` - Model comparison utilities

### Codex Skill (Equivalent)

**Main skill:** `codex/stratus/SKILL.md` (Codex-adapted)
- Same command catalog and outputs as Claude skill
- Codex usage notes for intent-triggered commands
- References the same docs, puzzles, and examples (mirrored)

**Supporting docs:** `codex/stratus/docs/` (5 files)
- `ARCHITECTURE.md` - X1-AC technical deep dive
- `BENCHMARKS.md` - Performance data and metrics
- `INTEGRATION.md` - Integration patterns and guides
- `API.md` - Complete API reference
- `TROUBLESHOOTING.md` - Debug and fix issues

**Puzzle suite:** `codex/stratus/puzzles/`
- `index.md` - All 20 impossible puzzles organized

**Code examples:** `codex/stratus/examples/` (4 files)
- `basic-reasoning.md`
- `rollout-planning.md`
- `agent-integration.md`
- `comparison.md`

### Codex Skill (Equivalent)

**Main skill:** `codex/stratus/SKILL.md` (Codex-adapted)
- Same command catalog and outputs as Claude skill
- Codex usage notes for intent-triggered commands
- References the same docs, puzzles, and examples (mirrored)

**Supporting docs:** `codex/stratus/docs/` (5 files)
- `ARCHITECTURE.md` - X1-AC technical deep dive
- `BENCHMARKS.md` - Performance data and metrics
- `INTEGRATION.md` - Integration patterns and guides
- `API.md` - Complete API reference
- `TROUBLESHOOTING.md` - Debug and fix issues

**Puzzle suite:** `codex/stratus/puzzles/`
- `index.md` - All 20 impossible puzzles organized

**Code examples:** `codex/stratus/examples/` (4 files)
- `basic-reasoning.md`
- `rollout-planning.md`
- `agent-integration.md`
- `comparison.md`

---

## Skill Capabilities

Once installed, the `/stratus` skill provides:

### Core Commands
```bash
/stratus                              # Overview + quick menu
/stratus help                         # Full command list
/stratus about                        # What is Stratus X1?
/stratus test                         # Quick diagnostic
```

### Reasoning (Chat Completions API)
```bash
/stratus ask [query]                  # Basic reasoning query
/stratus count [text]                 # Character counting
/stratus verify [statement]           # Logic verification
/stratus calculate [expression]       # Math calculation
/stratus analyze [data]               # Pattern analysis
```

### Planning (Rollout API)
```bash
/stratus plan [goal]                  # Let Stratus plan actions
/stratus validate [goal] [actions]    # Test action sequence
/stratus simulate [goal] [steps]      # Multi-step simulation
/stratus optimize [goal] [options]    # Compare action paths
```

### Puzzles (20 Impossible Tests)
```bash
/stratus puzzle list                  # Show all 20 puzzles
/stratus puzzle [number]              # Run specific puzzle
/stratus puzzle [number] [model]      # Compare with other model
/stratus puzzle benchmark             # Run full suite
```

### Comparison & Benchmarking
```bash
/stratus compare [query]              # Run on all 3 models
/stratus stats                        # Performance metrics
```

### Development & Debug
```bash
/stratus status                       # Server health
/stratus debug [query]                # Verbose output
/stratus diagnose                     # Full diagnostic
```

### Integration & Setup
```bash
/stratus integrate                    # Integration guide
/stratus models                       # Available models
/stratus quickstart                   # 60-second setup
/stratus examples                     # Code examples
```

---

## What is Stratus X1?

**Stratus X1 is a Predictive Action Model** - a Digital World Model for AI agents that predicts how digital environments respond to actions.

**Key characteristics:**
- **Not an LLM** - Operates in 768-dimensional embedding space
- **Predictive** - Predicts future states given current state + action
- **Self-supervised** - Learns from (state, action, next_state) tuples
- **Based on V-JEPA 2** - Meta's validated world model research
- **Production-ready** - 46,000 lines of code, comprehensive benchmarking

**Performance highlights:**
- 20-30× token reduction
- 2× task success rate (WebArena)
- <10ms latency per prediction
- 100% on impossible puzzles (vs 30-55% for LLMs)

---

## Quick Start

### 1. Install the Skill

```bash
cd ~/.claude/skills/
git clone https://github.com/formthefog/stratus-skills.git temp-stratus
cp -r temp-stratus/claude/stratus ./stratus
rm -rf temp-stratus
```

### 2. Set Environment Variables

```bash
export STRATUS_API_KEY=stratus_sk_live_...
export STRATUS_API_URL=http://212.115.124.137:8000
```

### 3. Test Installation

In Claude Code:
```
/stratus test
```

### 4. Try a Puzzle

```
/stratus puzzle 1
```

### 5. Compare Models

```
/stratus compare "Count r's in strawberry"
```

---

## For Developers

### Repository Structure

```
stratus-skills/
├── README.md                          # This file
├── AGENTS.md                          # Codex instructions for this repo
└── claude/
    └── stratus/                       # The complete skill
        ├── SKILL.md                   # Main skill (1,723 lines)
        ├── docs/                      # Supporting documentation (5 files)
        │   ├── ARCHITECTURE.md
        │   ├── BENCHMARKS.md
        │   ├── INTEGRATION.md
        │   ├── API.md
        │   └── TROUBLESHOOTING.md
        ├── puzzles/                   # Puzzle organization
        │   └── index.md
        └── examples/                  # Code examples (4 files)
            ├── basic-reasoning.md
            ├── rollout-planning.md
            ├── agent-integration.md
            └── comparison.md
└── codex/
    └── stratus/                       # Codex skill (equivalent)
        ├── SKILL.md
        ├── docs/
        ├── puzzles/
        └── examples/
```

### Integration Example

**OpenAI-compatible (2-line change):**

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://212.115.124.137:8000/v1",  # ← Change 1
    api_key="stratus_sk_live_..."                # ← Change 2
)

# Same API, now with Stratus world model
response = client.chat.completions.create(
    model="stratus-x1ac-small-claude-sonnet-4-5",
    messages=[{"role": "user", "content": "Count r's in strawberry"}]
)

print(response.choices[0].message.content)
# Output: "3" (correct, vs LLMs: "2" wrong)
```

---

## Troubleshooting

### Skill Not Found

**Check installation:**
```bash
ls ~/.claude/skills/stratus/SKILL.md
```

If missing, reinstall:
```bash
cd ~/.claude/skills/
git clone https://github.com/formthefog/stratus-skills.git temp-stratus
cp -r temp-stratus/claude/stratus ./stratus
rm -rf temp-stratus
```

### API Connection Error

**Verify environment variables:**
```bash
echo $STRATUS_API_KEY
echo $STRATUS_API_URL
```

If empty, add to `~/.bashrc` or `~/.zshrc`:
```bash
export STRATUS_API_KEY=stratus_sk_live_...
export STRATUS_API_URL=http://212.115.124.137:8000
```

Then reload:
```bash
source ~/.bashrc  # or source ~/.zshrc
```

### Skill Commands Not Working

**Restart Claude Code** after installation.

**Verify skill is recognized:**
```
/skills
```

Should show `stratus` in the list.

---

## Contributing

This is a Formation repository for Stratus X1 skills.

**To update the skill:**
1. Fork this repository
2. Make changes to `claude/stratus/`
3. Submit pull request
4. Formation team will review

**Issues:**
Report issues at: https://github.com/formthefog/stratus-skills/issues

---

## License

MIT License - See LICENSE file for details

---

## Contact

**Formation**
- Website: https://formation.ai
- GitHub: https://github.com/formthefog

**Stratus X1**
- Documentation: See `claude/stratus/docs/`
- API Reference: `claude/stratus/docs/API.md`

---

**This is the definitive Stratus X1 skill resource.**

Use it to understand, test, compare, integrate, and deploy the world's first production-ready predictive action model.
