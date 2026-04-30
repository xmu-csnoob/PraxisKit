---
name: plan-to-review
description: "Integrated PraxisKit execution entry point. Execute a compact plan or Kanban plan and produce work/review.md only when execution is explicitly authorized; otherwise dry-run."
---

# Plan to Review

Execute a PraxisKit plan and produce acceptance evidence:

```text
work/plan.md OR work/kanban.md + work/SUBAGENT.md -> plan-to-review -> code + work/review.md
```

This is a user-level harness over the execution semantics of `kanban-to-agents` and the acceptance packet semantics of `build-to-review`. Do not treat those skills as runtime functions. Reuse their scheduling, write-scope, validation, run-log, and review rules.

## Execution Authorization Invariant

This is a hard rule.

If the user asks to inspect, review, summarize, plan, or "see what's next", run **Dry-run mode** only.

Execution is authorized only when the user explicitly asks to execute, implement, advance, continue the plan, run agents, delegate, or parallelize.

Without explicit execution authorization, do not:

- edit project files
- mark tasks in progress or done
- spawn subagents
- install dependencies
- run destructive commands

Dry-run may read files and run non-mutating status or baseline inspection commands when local and reasonable.

## Inputs

Load the best available plan:

- `work/plan.md` for Light mode
- `work/kanban.md` + `work/SUBAGENT.md` for Standard/Full mode
- `work/praxiskit-context.md` if present
- `work/idea.md` and `work/PRD.md` when needed for acceptance evidence

If no plan exists, stop and recommend `shape-to-plan`.

## Modes

| Mode | Trigger | Behavior |
|---|---|---|
| Dry-run | No explicit execution authorization | Report next batch, blockers, write scopes, and proposed assignment. No edits. |
| Local | Critical path is small or write scopes overlap heavily | Orchestrator implements directly. |
| Hybrid | One blocking local task plus sidecar independent tasks | Orchestrator handles blocker; bounded sidecars may be delegated. |
| Parallel | Multiple independent tasks with disjoint write scopes | Spawn bounded workers, default max 3. |

## Dry-run Workflow

1. Read `work/plan.md` or `work/kanban.md` + `work/SUBAGENT.md`.
2. Parse task status:
   - `[ ]` pending
   - `[/]` in progress
   - `[x]` done
   - `[!]` blocked
3. Identify unblocked tasks from dependencies.
4. Report:
   - next executable batch
   - blockers
   - write scopes
   - local vs possible delegation assignment
   - validation commands that would run
5. Stop without edits.

## Execution Workflow

Run only when explicitly authorized.

1. **Check repo status.** Do not overwrite unrelated user changes.
2. **Establish baseline.** Read project scripts from `package.json`, `pyproject.toml`, `Cargo.toml`, `Makefile`, or README. Run local, reasonable baseline checks.
3. **Select executable batch.**
   - For `work/plan.md`, use `L{n}` task dependencies.
   - For `work/kanban.md`, use `T{wave}.{n}` dependencies.
4. **Choose execution mode.**
   - Keep immediate blockers local when waiting would stall the run.
   - Delegate only bounded sidecar work with disjoint write scopes.
   - Default to at most 3 parallel workers.
5. **Execute tasks.**
   - Mark selected tasks `[/]` only after work starts.
   - For delegated work, include plan/SUBAGENT path, exact task line, acceptance criteria, dependencies satisfied, owned files, and report format.
   - Tell workers they are not alone in the codebase and must not revert others' edits.
6. **Review and validate.**
   - Review returned changes before integration.
   - Run narrow validation first; broaden when shared behavior changed.
   - Mark `[x]` only after verification passes.
   - Mark `[!]` with reason when blocked.
7. **Write run log.** Create `work/agent-runs/YYYYMMDD-HHMM.md`.
8. **Update context.** Refresh `work/praxiskit-context.md`.
9. **Write review.** Create `work/review.md` using the review format below.

## Run Log Format

```markdown
# Agent Run: {timestamp}

## Scope
- Tasks:
- Mode:
- Baseline:
- Parallel Batch:

## Assignments
| Task | Agent | Write Scope | Parallel Batch | Result |
|------|-------|-------------|----------------|--------|

## Validation
- {command}: {result}

## Review
- Files reviewed:
- Scope compliance:
- Actual parallelism used:

## Follow-Ups
- {blocked task, failed check, or next wave}
```

## `work/review.md` Format

```markdown
# {Wave Review | Partial Build Review | Full Build Review | Regression Review}: {project or feature}

## Review Scope
- Type: Wave Review / Partial Build Review / Full Build Review / Regression Review
- Plan completion:
- User-inspectable artifact:
- Full-product acceptance allowed: Yes / No

## Original Promise
{1-3 sentences from plan/idea/PRD}

## What Is Ready To Inspect
- {artifact, URL, command, screenshot, file, or behavior}

## Cannot Inspect Because
- {missing UI, failed build, missing URL, incomplete wave, or not applicable}

## Demo Path
1. {step the user can take}
2. {expected visible result}
3. {value moment}

## Acceptance Match
| Source | Expectation | Result | Status |
|--------|-------------|--------|--------|
| Plan / PRD / Kanban | ... | ... | Pass / Gap / Unknown |

## Evidence
- Validation: `{command}` -> {result}
- Changed areas: `{path}`, `{path}`
- Run logs: `{path}`

## Gaps & Risks
| Gap / Risk | Impact | Plan Link | Recommended Next Step |
|------------|--------|-----------|-----------------------|

## Decision
- Accept this wave only:
- Revise completed work:
- Continue next plan batch:
- Do not accept full product yet:
```

If completion is below 100% or release gates fail, do not present the result as a fully accepted build.
