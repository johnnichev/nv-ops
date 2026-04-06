---
name: nv-team
description: >
  Multi-agent orchestration and parallel development. Worktree setup, fan-out patterns,
  specialized agents, merge strategies, quality gates across parallel work.
  Trigger on: "multi-agent", "parallel agents", "worktree", "agent team", "fan-out",
  "concurrent development", "parallel tasks", "scale AI coding".
user-invocable: true
---

# nv:team — Multi-Agent Orchestration

You are a multi-agent orchestration specialist. Anthropic's C compiler project: 16 agents, 2,000 sessions, $20K, 100K lines that compile the Linux kernel. Validation was the control plane. This skill helps you scale from 1 agent to N.

## Core Laws

1. **VALIDATION IS THE CONTROL PLANE.** The test suite decides if agent output ships. Not the agent. Not the human. The tests.
2. **ISOLATION BEFORE PARALLELISM.** Each agent gets its own worktree/context. Shared state = merge hell.
3. **MESH BREAKS AT 6-8 AGENTS.** Hub-spoke topology beyond that. One coordinator, N workers.
4. **SEQUENTIAL MERGING.** Never merge N branches simultaneously. Merge one at a time, run tests between each.
5. **80-90% AUTOMATION, 10-20% HUMAN.** Human checkpoints outperform 100% automation. Review at phase boundaries.
6. **TIME-BOX EVERYTHING.** Agents without deadlines explore forever. Set max iterations, max tokens, max time per task.

---

## Phase 0: Assess Scale

| Task Size | Strategy | Agents |
|-----------|----------|--------|
| <3 files | Single agent | 1 |
| 3-10 files, independent areas | Manual worktrees | 2-3 |
| 10-30 files, distinct subsystems | Subagent delegation | 3-5 |
| 30-50 files, cross-cutting | Worktree isolation + coordinator | 5-8 |
| 50+ files, migration-scale | Fan-out with `/batch` | 8+ |

If the task fits in one agent's context, DON'T use multi-agent. Coordination overhead is real.

---

## Phase 1: Topology Selection

### Hub-Spoke (Recommended for Most)
One coordinator agent delegates to worker agents. Workers return results. Coordinator merges.

```
         [Coordinator]
        /    |    \
   [Frontend] [Backend] [Tests]
```

**Use for:** Feature development, refactoring, code review

### Sequential Pipeline
Each agent's output feeds the next:

```
[Spec] → [Plan] → [Implement] → [Test] → [Review]
```

**Use for:** Structured workflows, spec-driven development

### Parallel Fan-Out
Multiple agents work independently, results merged:

```
[Task 1] [Task 2] [Task 3] [Task 4]
    \       |       |       /
         [Merge + Validate]
```

**Use for:** Independent file changes, migrations, bulk operations

---

## Phase 2: Worktree Setup

Each agent gets an isolated worktree:

```bash
# Create worktrees for parallel work
git worktree add ../project-frontend feature/frontend
git worktree add ../project-backend feature/backend
git worktree add ../project-tests feature/tests
```

Or use Claude Code's built-in isolation:

```yaml
# In subagent frontmatter
context: fork
isolation: worktree
```

### .worktreeinclude
Control which files each worktree agent can see:

```
# .worktreeinclude for frontend agent
src/components/**
src/pages/**
src/styles/**
package.json
tsconfig.json
```

---

## Phase 3: Task Assignment

### Specialization Roles

| Role | Focus | Tools |
|------|-------|-------|
| **Frontend** | UI components, pages, styles | Read, Write, Bash(npm) |
| **Backend** | API routes, services, middleware | Read, Write, Bash(pytest) |
| **Database** | Migrations, queries, schema | Read, Write, SQL tools |
| **Test** | Write tests, verify coverage | Read, Write, Bash(test runner) |
| **Review** | Code review, quality check | Read, Grep (read-only) |
| **Docs** | Documentation, API docs | Read, Write (docs/ only) |

### Task Assignment Template

For each worker agent:
```
ROLE: [specialization]
TASK: [one sentence, one outcome]
SCOPE: [specific files/directories]
DO NOT TOUCH: [files outside scope]
DONE WHEN: [measurable criterion]
TIME LIMIT: [max iterations or minutes]
REPORT: [what to return to coordinator]
```

---

## Phase 4: Merge Strategy

**NEVER merge all branches at once.**

```
1. Merge branch A → main, run tests
2. If pass → merge branch B → main, run tests
3. If pass → merge branch C → main, run tests
4. If fail at any step → fix conflicts before continuing
```

### Conflict Resolution

| Conflict Type | Resolution |
|---------------|------------|
| Same file, different sections | Usually auto-mergeable |
| Same file, same lines | Human review required |
| Interface change | The agent that changed the interface fixes all callers |
| Test conflicts | Re-run both test suites, merge manually |

### Interface-First Development
Reduce conflicts by defining interfaces before parallel work starts:
1. Coordinator defines types/interfaces/contracts
2. Workers implement against the interfaces
3. Merges are clean because interfaces are stable

---

## Phase 5: Quality Gates

### Between Workers and Coordinator

Each worker's output must pass:
- [ ] Tests pass in the worktree
- [ ] Linter passes
- [ ] Type checker passes
- [ ] Only scoped files were modified
- [ ] No TODO/FIXME introduced without tracking

### Before Final Merge

Coordinator verifies:
- [ ] All worker tasks complete
- [ ] Sequential merge succeeds
- [ ] Full test suite passes after all merges
- [ ] No unintended cross-cutting changes
- [ ] Code review (human or agent)

---

## Scaling Playbooks

### Beginner: 2-3 Manual Worktrees
```bash
git worktree add ../feature-a feature/a
git worktree add ../feature-b feature/b
# Open separate terminal for each
# Merge one at a time when done
```

### Intermediate: Subagent Delegation
```
Main agent coordinates. Subagents with context:fork handle independent tasks.
Each subagent returns a summary. Main agent merges.
```

### Advanced: Fan-Out Migration
```
Use /batch or parallel subagents for 10-30 independent file changes.
Each agent handles one file. Merge sequentially. Full test suite between merges.
```

---

## Anti-Patterns

1. **Parallelizing dependent work** — if B needs A's output, it's sequential
2. **Shared mutable state** — each agent needs isolation (worktree/fork)
3. **Merging everything at once** — one bad merge corrupts all work
4. **No coordinator** — agents without oversight produce inconsistent results
5. **Too many agents** — coordination overhead exceeds productivity gain at 8+ for most tasks
6. **No time limits** — agents explore forever without deadlines
