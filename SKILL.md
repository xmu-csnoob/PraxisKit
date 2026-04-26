---
name: prd-to-kanban
description: "Convert a PRD, design doc, or feature requirements into work/kanban.md and work/SUBAGENT.md for multi-agent implementation planning. Planning only. Sub-command: install for Claude Code trigger rules."
---

# PRD to Kanban Skill

Convert product requirements into a living markdown Kanban board and shared subagent context — readable by humans, editable by agents.

**Position in workflow:** PRD → **prd-to-kanban** → `kanban.md` → **subagent-driven-development** → implemented code.

This skill converts a static PRD into a living, agent-editable Kanban board. **You plan — you do not execute.** The generated `kanban.md` is ready for `subagent-driven-development` when the user wants execution. Do not implement tasks yourself.

## Install Sub-Command (`install`)

Invoke with `/prd-to-kanban install [--project]` to auto-configure trigger rules.

This sub-command is for Claude Code. When this skill is installed as a Claude plugin, the namespaced form may be `/prd-to-kanban:prd-to-kanban install [--project]`; behavior is the same.

Writes to `~/.claude/CLAUDE.md` (default) or `./CLAUDE.md` (with `--project`). Creates the file if missing. Idempotent — if the trigger row already exists, no change is made. If a `## Auto Mode — Required Skill Triggers` section already exists, the row is inserted without disturbing other entries.

In Codex, no install sub-command is needed after installing the skill or plugin. Codex discovers skills from installed skill/plugin metadata. If asked to install for Codex, provide the Codex skill/plugin marketplace installation steps instead of writing `CLAUDE.md`.

**Trigger block inserted:**
```markdown
## Auto Mode — Required Skill Triggers

When you detect any of the following, **invoke the corresponding skill immediately** — do not implement manually, do not ask first:

| Trigger | Skill |
|---------|-------|
| PRD, design doc, feature requirements, or "plan this" / "break this down" / "create a kanban" | `/prd-to-kanban` |

This is **not optional**. If you implement a planning task manually instead of invoking the skill, you are not following this workflow.
```

If the target file is not writable, report the error and suggest `! echo '...' >> ~/.claude/CLAUDE.md`.

## Trigger

Use this skill when:
- User provides a PRD, design doc, or feature requirements
- User says "plan this" or "break this down" or "create a kanban"
- Multi-agent or multi-phase development is anticipated
- The task touches more than 3 files or spans multiple modules

**Do NOT use for:**
- Single-file tasks or bug fixes (even with "plan this" language)
- Vague ideas without enough detail to decompose into tasks
- To-do lists with already-well-defined sequential tasks
- Non-implementation work (pure research, pure design without implementation intent)

If input is borderline, ask: "Does this need decomposition into independent tasks with owners and dependencies?" If no, decline politely.

## Input

Accept these forms:
1. Path to a PRD file (`.md`)
2. Raw requirements text pasted by user
3. Reference to an existing planning doc
4. Conversation context alone — infer tasks, epics, and dependencies from the current discussion

If input is vague, ask:
- "What is the target repository structure?" (for write-scope inference)
- "Are multiple agents/models involved?" (for role assignment)
- "Is there a frozen contract or schema that must not change?" (for dependency ordering)

## Output

Produce **two files** in the project's work directory:

| File | Purpose | Path |
|------|---------|------|
| **Kanban Board** | Human-readable + agent-editable status tracker | `<project>/work/kanban.md` |
| **Subagent Context** | Shared static context all subagents read at start | `<project>/work/SUBAGENT.md` |

`kanban.md` is dynamic (status changes). `SUBAGENT.md` is static (only updated when contracts change). Orchestrators pass the `SUBAGENT.md` path to every subagent — subagents read it themselves, no per-task slicing needed.

After writing both files, report their paths. Do not open files or launch external viewers unless the user explicitly asks.

---

## Output Format (`kanban.md`)

A single markdown file using native markdown syntax. No custom pipe-delimited formats, no HTML.

### Structure

```
# Kanban Board: {project} Phase {phase}
  > Source: {prd-file} · Agents edit this file · 📊 = computed
  > Last updated: {date} by {agent}

## Overview
| Metric | Value |
|--------|-------|
| Phase | {n} |
| Epics | {n}  |
| Tasks | {n}  |
| Done  | {n}  |
| Progress | {n}% |
| Agents | {n}  |
| Milestone | {id} — {title} |

## Milestones
| ID | Title | Status |
|----|-------|--------|
| M0 | ... | [ ] |
| M1 | ... | [/] |

## Epics
### E0: {title} [owner · {wave} · {milestone}] {done}/{total}
- [ ] T0.1 | {title} → {ac}
- [ ] T0.6 | {title} [T0.1,T0.2] → {ac}
- [ ] ⚠ T3.8 | {title} → {ac}

### E1: ...
...

## Dependencies & Parallelism 📊

### Dependency Layers (topological order)
| Layer | Tasks | Depends On | Wave |
|-------|-------|------------|------|
| L0 | T0.1, T0.2, ... | — | W0 |
| L1 | T1.1, T2.1 | L0 | W0..W1 |
| L2 | T1.3, T2.2 | L0..L1 | W1 |
| L3 | T1.5, T2.3, T3.4 | L0..L2 | W1 |

**Wave** = scheduling phase (time-based). **Layer** = topological position (structure-based).
Same layer + different owners + disjoint write scopes → safe to parallelize.

### Parallelism Windows (waves)
| Window | Wave | Tasks | Agents | Strategy |
|--------|------|-------|--------|----------|

(Group tasks by wave; one row per window.)

### Critical Path
`T0.2 → T3.1 → T3.2 → T3.4 → T5.1 → T5.5 → T5.6`

Longest dependency chain (computed). Tasks on this chain are marked with ⚠ in Epics.

## Risks & Gates
| Risk | Mitigation | Gate | Status |
|------|------------|------|--------|
| ...  | ...        | M0   | [ ]    |

## Agent Roles
| Role | Model | Strength | Best For |
|------|-------|----------|----------|
| Architect | ... | ... | ... |
```

### Syntax Rules

- **Status symbols**: `[x]` = done, `[ ]` = pending, `[/]` = in-progress, `[!]` = blocked
- **Blocked tasks** (`[!]`): waiting on external dependency, human decision, or failed attempt. Cannot be started even if resources are available.
- **Critical tasks**: Prefix title with `⚠` (e.g., `⚠ Wire integration`)
- **Inferred tasks**: Prefix title with `~` (e.g., `~ T0.7 | Add error handling`). Not in original PRD — add to epic tail, flag in Open Questions. Agents may not remove without author approval.
- **Dependencies**: In task line after title: `[T0.1,T1.2]`
- **Acceptance criteria**: After `→` at end of task line
- **Epic ownership** is a default — tasks inherit it unless overridden per-task
- **Computed fields** marked with 📊 — refresh on regeneration, never edit by hand

---

## Output Format (`SUBAGENT.md`)

A compact, shared context file read by every subagent at the start of its session. Static — set at generation time, only updated when contracts change. Keep it under 80 lines.

```
# SUBAGENT.md: {project}

## Project
{1-2 sentences: what it does, primary language/runtime}

## Stack
{comma-separated key deps}

## Contracts (frozen after W0 — do not modify)
- `{path}` — {what it defines/exports}

## Write Scopes
| Epic | Directory | Notes |
|------|-----------|-------|
| E0 | src/types/, src/db/ | Frozen at M0 |
| E1 | src/github/ | |
...

## Conventions
- Report back: `RESULT: {T_ID} | files: [...] | summary: {1 sentence}`
- Do not modify files outside your write scope
- Do not modify contract files (frozen)
```

Orchestrator passes `SUBAGENT.md` path + task description + dep outputs (dynamic, inline) to each subagent. Subagents read the file themselves — no per-task slicing by orchestrator needed.

---

## Task Granularity & Acceptance Criteria

A task should be:
- **Completable in one agent session** (1-3 hours of work max)
- **Self-contained** with a clear output (file created, function implemented, test added)
- **Max ~50 lines of code change** across files it touches

If a task would span multiple days or touch 5+ files, **break it down**. If it's a single checkbox (e.g., "fix bug X in Y"), it's too small — handle inline.

AC must be **verifiable by an agent** (not subjective).

| Task Type | Size Target | Decompose If | Good AC Pattern |
|-----------|-------------|--------------|-----------------|
| Schema / contract / type | 1-3 files | 5+ new types without grouping | "Types compile; schema validation passes with valid + invalid inputs" |
| API / endpoint | 1-3 files per endpoint | Endpoint references 3+ services | "GET /foo returns 200 with expected shape; 401 unauthenticated" |
| UI / component | 1 file per component | References 3+ unrelated areas | "Renders with props X, Y, Z; passes snapshot test; keyboard-navigable" |
| Feature / logic | 2-4 files | Touches 5+ modules | "Given X, when Y, then Z (unit test covers path)" |
| Test | 1 test file per feature | Setup in 3+ separate areas | "Test file runs; N/M assertions cover key paths; no false positives" |
| Integration / wiring | 1 file | Depends on 5+ tasks | "End-to-end flow completes; no unhandled errors in logs" |
| Config / infra | 1-2 files | — | "Service starts; config loaded from env; validation errors surface" |

If AC cannot be made concrete, mark task `Needs Human Approval` with `~` as AC placeholder. In multi-agent runs with no human in the loop: mark `[!]` blocked and add to **## Open Questions**.

---

## Execution Rules

1. **Parse the PRD** to extract all tasks, epics, dependencies, and milestones
2. **Apply task granularity checks** — verify each task is completable in one session, <~50 lines change. Break down anything too large; skip anything too small
3. **Write acceptance criteria** using the DoD table. Flag unresolvable ACs as `Needs Human Approval` with `~`
4. **Compute topological layers** from dependency graph — same layer = no transitive deps
5. **Identify the critical path** — longest chain through dependency graph
6. **Assign owners** — epic header owner is a default; individual tasks inherit unless overridden. Integration/wiring tasks always have a single explicit owner
7. **Determine parallelism opportunities** — same layer + different owners + disjoint write scopes = parallel. Build dependency layers table and parallelism windows table
8. **Generate `kanban.md`** and report the file path. Reflect actual in-progress state — do not reset existing `[x]`/`[/]` to pending
9. **Generate `SUBAGENT.md`**: project summary, stack, frozen contracts (paths + what they define), write scope table per epic, conventions. Keep it under 80 lines.
10. **Handoff.** Both files are ready for `subagent-driven-development`. Invoke it if the user wants immediate execution; otherwise, present the board and wait.

---

## Execution Notes

- If PRD has **pre-existing dependency info**, preserve it in task-level `[deps]`
- Always suggest **worktree or branch isolation** when parallel agents are involved
- **Orchestrator owns kanban updates** — subagents implement and report back; only the orchestrator edits task statuses to avoid concurrent write conflicts

If unresolved questions exist during generation, add an `## Open Questions` table (`Task | Blocked By | Question`). When answered, move to `## Resolved Questions` with answer inline.
