# Auto Methodology

**Let AI run 100+ experiments overnight while you sleep.**

A meta-skill for Claude Code that turns any measurable goal into an autonomous optimization loop. Based on the pattern Andrej Karpathy used in [AutoResearch](https://github.com/karpathy/autoresearch) -- where a single AI agent ran 700 experiments and discovered optimizations that a world-class ML researcher missed.

```
Human writes program.md    AI runs experiments    AI evaluates    AI keeps winners
        |                        |                     |                |
        v                        v                     v                v
   "Optimize X"  ──>  modify train.py  ──>  measure val_bpb  ──>  git commit / revert
                            ^                                          |
                            └──────────────────────────────────────────┘
                                         loop forever
```

---

## The Core Idea

Every optimization problem -- no matter the domain -- can be reduced to **three primitives**:

| Primitive | What it is | Example |
|-----------|-----------|---------|
| **Editable Asset** | The one file the agent can modify | `train.py`, `queries.sql`, `prompts.md` |
| **Scalar Metric** | A single number: better or worse, no ambiguity | `val_bpb`, `p95_latency_ms`, `lighthouse_score` |
| **Time-boxed Cycle** | Fixed duration per experiment for fair comparison | 5 min wall-clock time |

Map your problem to these three, and the agent handles the rest -- proposing changes, measuring results, keeping wins, reverting losses, logging everything.

## What This Skill Does

This is not a tool that optimizes something specific. It's a **tool that builds optimization tools**.

Tell it what you want to improve, and it generates:

```
auto-[your-domain]/
  program.md          # Research charter: goals, constraints, rules
  [editable-asset]    # The file the agent experiments on
  evaluate.sh         # Immutable scoring logic
  run_loop.sh         # Ratchet loop with timeout + logging
  results.tsv         # Append-only experiment ledger
```

Then point any coding agent at `program.md` and let it run.

## Real-World Results

| Project | What was optimized | Experiments | Result |
|---------|-------------------|-------------|--------|
| Karpathy AutoResearch | LLM training efficiency | 700 | 11% faster training, discovered QK-Norm missing multiplier |
| Shopify Liquid | Template rendering | ~100 | 53% faster rendering, 61% less memory |
| SkyPilot (16 GPU parallel) | LLM training | 910 in 8hrs | Massive exploration at scale |

## 6 Archetypes

The skill matches your problem to the right loop pattern:

```
AutoResearch    hypothesis ──> experiment ──> measure ──> keep/discard
AutoBuild       code change ──> test/benchmark ──> keep/revert
AutoSearch      search ──> filter ──> rank ──> synthesize
AutoOperate     check state ──> classify ──> act ──> verify
AutoContent     draft ──> score with rubric ──> revise ──> keep best
AutoAnalysis    candidate explanation ──> test against evidence ──> promote strongest
```

## 4-Surface Architecture

Strict separation of concerns prevents the agent from gaming the system:

```
┌─────────────────────────────────────────────────┐
│  PROGRAM SURFACE (human-controlled)             │
│  program.md: goals, constraints, promotion rules│
├─────────────────────────────────────────────────┤
│  MUTABLE SURFACE (agent-controlled)             │
│  train.py / queries.sql / prompts.md            │
├─────────────────────────────────────────────────┤
│  ORACLE SURFACE (immutable)                     │
│  evaluate.sh, test data, scoring functions      │
├─────────────────────────────────────────────────┤
│  LEDGER SURFACE (append-only)                   │
│  results.tsv: every run logged, wins and losses │
└─────────────────────────────────────────────────┘
```

The agent **cannot modify the oracle**. If it could, it would optimize the scoreboard instead of the actual problem.

## The Ratchet

Progress is monotonic. The Git-based ratchet ensures the codebase only moves forward:

```
  metric improves + constraints met     ──>  keep commit (new baseline)
  metric equal + solution simpler       ──>  keep commit
  metric regresses                      ──>  git reset --hard HEAD~1
  experiment crashes                    ──>  log error, revert, continue
  evidence ambiguous                    ──>  mark "hold", move on
```

## 8 Domain Examples

The skill ships with detailed examples for:

| Domain | Editable Asset | Metric | Cycle |
|--------|---------------|--------|-------|
| ML Training | `train.py` | val_bpb | 5 min |
| Web Performance | `index.html` + inline assets | Lighthouse score | 2 min |
| SQL Optimization | `queries.sql` | P95 query time (ms) | 1 min |
| Prompt Engineering | `prompts.md` | F1 on labeled set | 3 min |
| Build Optimization | `CMakeLists.txt` | Full build time (s) | 60 min |
| Content/Copy | `landing_copy.md` | Composite quality score | 1 min |
| API Performance | `handler.go` | P99 latency (ms) | 3 min |
| LLM Inference | `serve_config.py` | tokens/sec (quality-gated) | 5 min |

See [`references/domain-examples.md`](references/domain-examples.md) for full configs with evaluation scripts and exploration directions.

## 5 Guardrails

Hard rules that keep the loop honest:

1. **Can't measure it? Don't optimize it.** Establish automated scoring before starting.
2. **No moving the goalposts.** Metrics and promotion rules are locked before the loop starts.
3. **Don't scale what doesn't work.** Single loop must be stable before adding parallelism or orchestration.
4. **More autonomy =/= better methodology.** Human checkpoints at appropriate intervals based on risk.
5. **No orchestration before validation.** One agent, one loop, one metric must produce value first.

## Installation

```bash
# Clone and install
git clone <this-repo>
claude skill install ./auto-methodology

# Or copy the skill directory into your project's .claude/skills/
cp -r auto-methodology /path/to/project/.claude/skills/
```

## Usage

Just describe what you want to optimize. The skill activates automatically:

```
> "I want to automatically optimize my SQL query performance"
> "Help me set up an overnight experiment loop for prompt engineering"
> "Build me an AutoBuild system for my API latency"
```

The skill will walk you through:

1. Clarifying your goal and baseline
2. Identifying the three primitives
3. Choosing an archetype
4. Generating all the files
5. Pre-launch checklist
6. Launch and monitoring guidance

## Project Structure

```
auto-methodology/
  SKILL.md                          # The skill definition (core logic)
  README.md                         # You are here
  references/
    program-template.md             # Fill-in-the-blank template for program.md
    domain-examples.md              # 8 domain examples with full configs
```

## When NOT to Use This

Not every problem benefits from autonomous loops:

- **Requires human creativity** -- design aesthetics, brand voice
- **Evaluation is expensive** -- needs real user feedback or week-long A/B tests
- **Tiny solution space** -- only 2-3 possible configurations
- **High-risk mutations** -- production database schemas, critical infrastructure

## Philosophy

> "You're no longer writing the Python file. You're writing program.md -- you're programming the program."
> -- Andrej Karpathy

The paradigm shift:

| Era | Who does what |
|-----|--------------|
| Traditional | Human writes code, human tests, human improves |
| Vibe Coding | Human describes, AI writes, human reviews |
| **Auto** | **Human sets direction, AI experiments, AI evaluates, AI improves, human checks occasionally** |

You don't need to know the optimal learning rate. You just need to know that "learning rate is worth exploring." The agent will find the optimum -- and things you never thought to try.

## License

MIT
