---
name: task-graph-to-batch
description: "Select unblocked tasks from work/task-graph.md and write a work/execution-batch-{n}.md artifact for the next wave. Planning only; does NOT execute tasks. Hand off to batch-to-build for execution."
---

# Task Graph to Batch

Pick the next executable batch from a task graph. This is half of the v2-M `kanban-to-agents` skill — the scheduling half. Execution is now in `batch-to-build`.

```text
work/task-graph.md -> task-graph-to-batch -> work/execution-batch-{n}.md -> batch-to-build
```

## Contract

**Inputs:** `task_graph` (`work/task-graph.md` + `work/SUBAGENT.md`)
**Output:** `execution_batch` (`work/execution-batch-{n}.md`)
**Schema:** `schemas/execution-batch.schema.md` (v3.0)
**Preconditions:**
- `work/task-graph.md` exists and follows `schemas/task-graph.schema.md`
- `work/SUBAGENT.md` exists
- Project baseline has been checked. If it fails, create only a baseline-repair batch for `T0.0 | Restore build baseline`
- At least one task with status `[ ]` and all dependencies `[x]`
**Postconditions:**
- `work/execution-batch-{n}.md` lists selected tasks with parallel groups identified
- Disjoint write scopes within each parallel group
- Any 2+ task parallel group records `execution_mode: subagent-driven` and one subagent dispatch expectation per task
- Authorization defaults to `dry-run` unless the user explicitly authorizes implementation with a narrow execution phrase or host-native execution decision for this exact batch
- The batch artifact is self-contained enough for `batch-to-build` to execute without rereading the PRD or full task graph
**Clarification gate:** does NOT fire. Pre-flight-validated.
**Side effects:**
- Writes `work/execution-batch-{n}.md`
- Reads but does NOT modify `work/task-graph.md`
- May update `work/praxiskit-context.md`
**Stop boundary:** Does NOT execute tasks, spawn agents, create source directories, install dependencies, or modify project source code. Hands off to `batch-to-build`.

## Inputs

Require these unless the user provides equivalent content:

- `work/task-graph.md` — task list with statuses and dependencies
- `work/SUBAGENT.md` — frozen contracts and write scopes
- `work/praxiskit-context.md` — optional cross-skill index

If either required file is missing, stop and recommend invoking `prd-to-task-graph` (or `seed-to-task-graph` for light recipe).

## Preflight

Before selecting tasks, establish baseline health:

- Read project scripts from `package.json`, `pyproject.toml`, `Cargo.toml`, `Makefile`, or README
- Identify the narrowest test command and build/typecheck command
- Run baseline checks when they are local and reasonable
- Record failures as pre-existing blockers. If a baseline failure blocks release, write a baseline-repair batch with `T0.0 | Restore build baseline` as the only sequential task, `baseline.status = fail`, and `baseline_repair = true`
- If no baseline command exists, write `baseline.status = unavailable`, keep authorization in dry-run mode, and note that execution is blocked until a baseline command is added and rechecked to `pass`
- Do not attribute baseline failures to the current wave

## Selection Rules

1. Parse task status: `[ ]` pending, `[/]` in progress, `[x]` done, `[!]` blocked.
2. A task is unblocked only when all dependencies in `[T...]` are `[x]`.
3. Prefer the earliest dependency layer and wave.
4. Identify critical-path tasks; mark them sequential if they have shared write scope with other selected tasks.
5. Group tasks with disjoint write scopes into parallel groups. Never put two tasks into the same parallel group if their write scopes overlap or if either touches a frozen contract.
6. Limit `max_parallel` to 3 unless the user requests more.

## Authorization

The batch artifact is always written, but with `authorization: dry-run` by default.

Execution authorization can be collected in either of these ways:

1. **Host-native execution decision** after the batch is generated.
   - Preferred when the user is present.
   - Ask: "Execute Batch {n} now?"
   - Options:
     - `execute_now` — set `authorization: execute` and hand off to `batch-to-build`
     - `keep_dry_run` — leave dry-run and stop
     - `revise_batch` — leave dry-run and ask what should change
   - Include the exact batch path, task IDs, parallel groups, subagent dispatch expectation, and validation commands in the decision prompt so the authorization is scoped to this batch and execution mode.
2. **Narrow execution phrase** in the current user turn.

These phrases authorize execution:
- "execute this batch"
- "implement this wave"
- "modify code for this batch"
- "run agents for this batch"
- "delegate implementation for this batch"

If the user authorizes by decision or phrase in the current turn:
- Set `authorization: execute` in the batch artifact
- Record the authorization timestamp
- Record the authorization source: `decision-ui`, `chat-confirmation`, or `phrase`
- If any parallel group contains 2+ tasks, record that execution is `subagent-driven` and that the orchestrator must dispatch one subagent per task before doing implementation work.

Otherwise, the batch is dry-run only. Words like "advance", "continue", "next", or "proceed" are ambiguous and MUST NOT set `authorization: execute` by themselves.

## Execution Phrase Routing

If the current user turn is one of the exact execution phrases above, or the user selects `execute_now` in the host-native execution decision:
- If a current `work/execution-batch-{n}.md` already exists and matches the next step in `work/praxiskit-context.md`, do not generate another batch. Route to `batch-to-build` for that batch.
- If no current batch exists and this skill must select one, write the new batch with `Authorization: execute`, `Approved by user: yes`, and the current timestamp.
- Still do not modify source code in this skill. Execution belongs to `batch-to-build`.

## Workflow

1. Check repo status and run preflight.
2. Read task-graph.md and parse statuses + dependencies.
3. Select unblocked tasks for the next wave.
4. Compute parallel groups and sequential tasks.
5. Write `work/execution-batch-{n}.md` per `schemas/execution-batch.schema.md`, including:
   - task graph fingerprint (mtime or hash)
   - exact selected task rows copied from `work/task-graph.md`
   - exact acceptance criteria for each selected task
   - selected task dependencies
   - selected task status at batch generation time
   - owned write scopes and frozen-contract paths relevant to the selected tasks
   - validation command(s) selected during preflight
   - execution mode (`subagent-driven` when any parallel group has 2+ tasks)
   - explicit subagent dispatch expectations for every parallel group
6. Surface the execution decision if the batch is still dry-run and the user is present. Prefer host-native decision/input UI; use one direct chat question only if no structured input is available. Do not force the user to type a magic phrase after a batch was just generated. If the batch has a parallel group, the prompt must state that execution will be subagent-driven.
7. If the user authorizes execution, update the batch authorization block to `Mode: execute`, `Approved by user: yes`, `Authorization source: decision-ui` (or `chat-confirmation`), and current timestamp, then hand off to `batch-to-build`.
8. Update `work/praxiskit-context.md` with the batch path and authorization state.
9. Report the generated path. State whether execution was authorized now or remains dry-run.

## Planning-Only Guardrail

During this skill, do not create source directories, scaffold files, install packages, run formatters, or make any source-tree changes for the selected tasks. Creating directories such as `src/components/...` counts as implementation and belongs only in `batch-to-build` after authorization.
