# nv:ops

**Agent infrastructure: safety, measurement, scaling.**

```bash
npx skills add johnnichev/nv-ops -g -y
```

3 skills for running AI coding agents safely at scale:

| Skill | Command | What it does |
|-------|---------|--------------|
| **nv:guard** | `/nv-guard` | Guardrails: permission tiers, destructive action blocking, secret remediation, auto-checkpoints, graduated trust (L0-L3), audit trails. "Make all mistakes recoverable." |
| **nv:eval** | `/nv-eval` | Agent evaluation: config quality scoring (A-F grading), Quality Flywheel, before/after measurement, compound failure math. |
| **nv:team** | `/nv-team` | Multi-agent orchestration: hub-spoke topology, worktree isolation, sequential merging, validation as control plane. |

## Why You Need This

Real AI agent incidents:
- Replit wiped a production database
- Claude ran `rm -rf ~/`
- Cursor deleted 70 files in Plan Mode
- Google Antigravity deleted an entire D: drive

The fix isn't "prevent all mistakes." It's **"make all mistakes recoverable."**

nv:ops sets up the infrastructure that makes agent autonomy safe: bootstrap templates for permissions/hooks, auto-checkpoints before every edit, destructive action blocking, secret remediation, graduated trust levels.

## Research-Backed

| Finding | Source |
|---------|--------|
| 85% per-step accuracy = 20% success over 10 steps (compound failure) | OpenAI Agents docs |
| Anthropic's C compiler: 16 agents, 2,000 sessions, 100K lines of code | Anthropic engineering blog |
| Mesh topology breaks at 6-8 agents — use hub-spoke for more | Multi-agent research |
| Validation is the control plane (not the agent, not the human — the tests) | Anthropic C compiler |

## Install

```bash
npx skills add johnnichev/nv-ops -g -y
```

Installs all 3 skills globally. Works with Claude Code, Cursor, GitHub Copilot, Windsurf, Aider, Gemini CLI, and 9+ other tools.

## Use

```
/nv-guard     # Set up safety infrastructure (do this first)
/nv-eval      # Score and improve agent configs
/nv-team      # Coordinate multiple agents on parallel tasks
```

## See Also

- **[nv:context](https://github.com/johnnichev/nv-context)** — Set up context engineering for your repo (the foundation)
- **[nv:dev](https://github.com/johnnichev/nv-dev)** — Plan, test, debug workflow
- **[nv:design](https://github.com/johnnichev/nv-design)** — Professional web design with AI

## License

MIT

---

**Built by [johnnichev](https://github.com/johnnichev).** For engineers who ship.
