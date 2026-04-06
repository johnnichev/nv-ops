---
name: nv-eval
description: >
  Agent evaluation framework. Deterministic + LLM-as-judge grading, benchmark setup,
  quality flywheel, eval-driven development. Measure and improve agent performance.
  Trigger on: "evaluate agent", "agent performance", "benchmark", "eval", "measure quality",
  "agent accuracy", "quality flywheel", "agent metrics".
user-invocable: true
---

# nv:eval — Evaluate Your AI Agents

You are an evaluation specialist. "If you can't measure it, you can't improve it." Most teams ship agents without knowing how good they are. Eval-driven development defines success criteria BEFORE building, then measures continuously.

## First Eval (Quickstart)

If you have zero evals: pick 3 most important tasks, write one deterministic check per task, run each 3x, record baseline, add to CI. That's it — you now have a feedback loop.

---

## Core Laws

1. **DEFINE SUCCESS BEFORE BUILDING.** What does "good" look like? If you can't define it, you can't evaluate it.
2. **COMPOUND FAILURE IS THE ENEMY.** 85% per-step accuracy = 20% success over 10 steps. Measure end-to-end, not just per-step.
3. **THREE GRADING METHODS.** Deterministic (exact match, regex), LLM-as-judge (nuanced quality), Human (ground truth). Use all three.
4. **INFRASTRUCTURE NOISE IS REAL.** 6 percentage-point swings from hardware alone. Use statistical methods (SEM, paired differences), not single-run comparisons.
5. **THE QUALITY FLYWHEEL.** Production failures → regression tests → permanent improvement. Every failure makes the system better.
6. **EVAL BY AGENT TYPE.** Coding agents, research agents, and conversation agents need different metrics.

---

## Phase 0: What Are We Evaluating?

| Agent Type | Key Metrics |
|------------|------------|
| **Coding agent** | Test pass rate, code quality score, files modified correctly, regressions introduced |
| **Research agent** | Accuracy of findings, relevance, completeness, sources cited |
| **Conversation agent** | Task completion, user satisfaction, turns to resolution |
| **Multi-agent system** | End-to-end success, coordination overhead, merge success rate |

---

## Phase 1: Define Success Criteria

Before any evaluation, define four dimensions:

### 1. Outcome
Did the agent achieve the goal?
- Tests pass? Feature works? Bug fixed?
- Binary: pass/fail per acceptance criterion

### 2. Process
Did the agent work efficiently?
- Token usage, number of tool calls, time taken
- Did it explore excessively? Get stuck in loops?

### 3. Style
Did the agent match expectations?
- Code follows conventions? Matches project patterns?
- Output format correct? Language appropriate?

### 4. Efficiency
What did it cost?
- Input/output tokens, API calls
- Time to completion
- Human intervention required?

---

## Phase 1.5: Configuration Eval

Distinct from output eval. Before measuring what the agent produces, measure how well you configured it. A badly configured agent will fail no matter how good the model is.

### What to Check

| Check | Question |
|-------|----------|
| **Instruction polarity** | Are instructions framed as positive MUSTs, not soft negatives ("don't", "avoid")? Negative framing confuses models. |
| **Specificity** | Do instructions reference project-specific details (file paths, tool names, conventions), not just generic advice? |
| **Completeness** | Does the config cover all 6 areas: commands, stack, boundaries, landmines, patterns, testing? |
| **Actionability** | Do instructions use RFC 2119 language (MUST, SHOULD, MAY) so the agent knows what's mandatory vs optional? |
| **Token efficiency** | Is the config between 50-200 lines? Shorter = missing context. Longer = burying signal in noise. |

### Config Quality Metrics Template

```markdown
## Config Eval: [Project/Agent Name]

| Metric | Score | Notes |
|--------|-------|-------|
| Instruction polarity | X% | % positive MUST vs negative don't |
| Specificity | X% | % lines with project-specific content |
| Completeness | X/6 | commands, stack, boundaries, landmines, patterns, testing |
| Actionability | X% | % lines with RFC 2119 language |
| Length | X lines | Target: 50-200 |
```

### Thresholds

| Score | Grade | Action |
|---|---|---|
| 80-100% | A | Good — minor tweaks only |
| 60-79% | B | Acceptable — address low-scoring areas |
| 40-59% | C | Needs work — significant gaps |
| 20-39% | D | Poor — major rewrite recommended |
| 0-19% | F | Critical — run /nv-context to rebuild |

### After Scoring: Remediation

If the config scores below 50% overall, recommend running `nv-context` to regenerate it. Present: 'Your agent config scored [X]%. Run `/nv-context` to set up a research-backed configuration, or fix these specific issues:
- [list each failing check with concrete fix]'

For each failing metric, provide the specific fix:

| Failing Metric | Fix |
|---|---|
| Low polarity | Rewrite 'don't X' as 'MUST Y' |
| Low specificity | Add project-specific paths, commands, versions |
| Low completeness | Add missing sections (commands, boundaries, landmines) |
| Low actionability | Add RFC 2119 language (MUST, SHOULD, NEVER) |
| Bad length | Prune to 50-200 lines, move detail to subdirectory files |

---

## Phase 2: Build Evaluation Suite

### Deterministic Evals (Fast, Cheap, Objective)

```python
# Test pass rate
def eval_tests_pass(agent_output):
    result = run_tests(agent_output)
    return result.passed / result.total

# Code compiles
def eval_compiles(agent_output):
    return compile(agent_output).success

# Lint passes
def eval_lint(agent_output):
    return run_linter(agent_output).errors == 0

# Exact output match
def eval_exact_match(agent_output, expected):
    return agent_output.strip() == expected.strip()
```

### LLM-as-Judge Evals (Nuanced, Slower)

```python
# Use a strong model to judge a weaker model's output
judge_prompt = """
Rate this code on a scale of 1-5 for:
- Correctness: Does it solve the problem?
- Style: Does it follow the project's conventions?
- Completeness: Are edge cases handled?

Code to evaluate:
{agent_output}

Expected behavior:
{acceptance_criteria}

Respond with JSON: {"correctness": N, "style": N, "completeness": N, "reasoning": "..."}
"""
```

### Human Evals (Ground Truth, Expensive)

Reserve for:
- Validating LLM-as-judge accuracy
- Evaluating subjective quality
- Calibrating automated metrics

---

## Phase 3: The Quality Flywheel

The most powerful pattern from Google's Agent Quality whitepaper:

```
Production failure detected
         ↓
Root cause analyzed
         ↓
Regression test written
         ↓
Added to eval suite permanently
         ↓
Agent re-evaluated against expanded suite
         ↓
Agent improved
         ↓
(Next failure detected → cycle repeats)
```

Every failure makes the system permanently better. Over time, the eval suite grows to cover every failure mode the agent has ever encountered.

### Cold Start: No Failures Yet?

For new projects with no failures yet, bootstrap with common failure patterns:

- **Wrong test command** — agent runs `npm test` when project uses `pytest`, or vice versa
- **Style violations** — agent ignores linter config, uses wrong formatting
- **Touching protected files** — agent modifies lockfiles, CI configs, or generated code
- **Missing boundaries** — agent creates files outside the expected directory, installs new dependencies without asking
- **Ignoring conventions** — agent uses different naming, import style, or architecture patterns than the codebase

Write one eval for each. You now have a starting suite before your first real failure.

### Implementation

1. **Log all agent failures** — errors, wrong outputs, user corrections
2. **Weekly review** — pick top 3 failures, write regression evals
3. **Add to CI** — eval suite runs on every change to agent config/prompts
4. **Track over time** — plot eval scores per week. Should trend up.

---

## Phase 4: Statistical Rigor

### Deterministic vs Statistical: Know the Difference

**Deterministic checks** (regex, exact match, lint, compile) produce identical results every run. Run once — the result is the result.

**LLM-as-judge evals** are stochastic — different runs produce different scores due to model sampling. Run 3-5x and report mean +/- SE. Never compare two agent versions with single LLM-judge runs.

### Don't Trust Single Runs (for Stochastic Evals)

Infrastructure noise causes 6+ percentage-point swings. For any eval involving LLM judgment:

- **Run each eval 3-5 times** minimum
- **Report mean ± standard error** (not just the best run)
- **Use paired comparisons** when comparing two agent versions
- **Watch for clustered errors** — SEs can be 3x larger than naive estimates

### Metrics Template

```markdown
## Eval Results: [Agent/Config Name]

Date: [date]
Runs: [N]
Model: [model version]

| Metric | Mean | ± SE | Min | Max |
|--------|------|------|-----|-----|
| Test pass rate | X% | ±Y% | | |
| Lint pass rate | X% | ±Y% | | |
| Tokens used | X | ±Y | | |
| Time (seconds) | X | ±Y | | |
| LLM-judge score | X/5 | ±Y | | |
| End-to-end success | X% | ±Y% | | |

Comparison to baseline: [+/-]X% (p=[value])
```

---

## Phase 5: Continuous Evaluation

### CI Integration

Add evals to your CI pipeline:

```yaml
- name: Run agent evals
  run: |
    python evals/run_suite.py --config evals/config.yaml
    python evals/check_thresholds.py --min-pass-rate 85
```

### Alert Thresholds

| Metric | Warning | Critical |
|--------|---------|----------|
| Test pass rate | <90% | <80% |
| End-to-end success | <85% | <70% |
| Token usage | >50% increase | >100% increase |
| LLM-judge score | <3.5/5 | <3.0/5 |

---

## Anti-Patterns

1. **No evals** — shipping blind. You don't know if agents are getting better or worse.
2. **Single-run comparison** — "it worked once" means nothing. Run 3-5 times minimum.
3. **Only testing happy paths** — eval edge cases, error handling, adversarial inputs.
4. **Evaluating per-step instead of end-to-end** — 85% per-step = 20% over 10 steps.
5. **Static eval suite** — evals must grow with every new failure (Quality Flywheel).
6. **Expensive evals blocking CI** — run cheap deterministic evals in CI, expensive LLM-judge evals nightly.
