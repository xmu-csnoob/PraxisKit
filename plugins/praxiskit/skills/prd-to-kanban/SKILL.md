---
name: prd-to-kanban
description: "Convert a PRD, design doc, feature requirements, or PraxisKit work/PRD.md into work/kanban.md and work/SUBAGENT.md for dependency-aware multi-agent implementation planning. Planning only; do not implement tasks."
---

# PRD to Kanban

Position in PraxisKit:

```text
work/PRD.md -> prd-to-kanban -> work/kanban.md + work/SUBAGENT.md -> kanban-to-agents
```

## Pre-flight

### Blocking Check

Before decomposing any tasks, load `schemas/kanban.schema.md` and run these checks. If any check fails, **stop immediately** -- do not write kanban.md.

### Check 1: No Blocking Open Questions

Scan `work/PRD.md` -> `## Open Questions` table. Find rows where `Blocks` = `implementation` or `release` AND the question is unresolved.

If any exist, output:

```
Cannot generate kanban: the following blocking Open Questions must be resolved first.

Blocking questions:
1. [{Question}] -- blocks: {Blocks value}
2. ...

Resolution options:
- Re-run `idea-to-prd` to answer these questions
- Or answer them directly and update work/PRD.md, then retry
```

Stop. Do not write kanban.md.

### Check 2: All FRs Have Acceptance Criteria

Scan `## Functional Requirements` table. Find FR rows where `Acceptance Criteria` is empty or vague (contains "TBD", "TODO", or is blank).

If any exist, output:

```
Cannot generate kanban: these FRs have missing acceptance criteria:
- {FR_ID}: {Requirement}
...
Add Given/When/Then criteria to these rows in work/PRD.md, then retry.
```

### Check 3: At Least 1 Milestone

Check that `## Milestones` has at least one row. If not, output:

```
Cannot generate kanban: work/PRD.md has no milestones defined.
Add at least one milestone to ## Milestones, then retry.
```

## Main Workflow

After all pre-flight checks pass, invoke the shared `prd-to-kanban` skill and follow its instructions exactly. That skill is the source of truth for output format, task granularity, acceptance criteria, dependency layering, critical path, and subagent handoff rules.

If the shared source is unavailable, use this fallback:

- Convert the PRD into `work/kanban.md` and `work/SUBAGENT.md`
- Plan only. Do not implement tasks.
- Use verifiable acceptance criteria for every task (copy from PRD FRs)
- Put frozen contract/schema work in Wave 0
- Compute dependency layers, parallelism windows, and the critical path
- Keep `SUBAGENT.md` under 80 lines
- Report the generated paths and wait unless the user asks for execution
