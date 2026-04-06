# nv:ops

Agent infrastructure skills for AI coding agents. Safety, measurement, and scaling.

## Skills

- `nv-guard` — Guardrails and safety. Bootstrap template for permissions/hooks, destructive action blocking, secret remediation, auto-checkpoints, graduated trust (L0-L3), audit trails. "Make all mistakes recoverable."
- `nv-eval` — Agent evaluation. Config quality scoring (polarity, specificity, completeness, actionability, length), A-F grading, remediation handoff to nv-context, Quality Flywheel, before/after measurement.
- `nv-team` — Multi-agent orchestration. Hub-spoke/pipeline/fan-out topologies, worktree isolation, specialized roles, sequential merging, quality gates. Validation is the control plane.

## Install

```bash
npx skills add johnnichev/nv-ops -g -y
```

## Boundaries

### Always
- MUST keep each SKILL.md under 500 lines
- MUST use RFC 2119 language

### Never
- NEVER remove research-backed claims without replacement data
- NEVER merge skills into a single SKILL.md — one file per skill
- NEVER auto-write to .claude/ (bootstrap template pattern only)
