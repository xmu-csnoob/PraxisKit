---
name: shape-to-plan
description: "Integrated PraxisKit planning entry point. Turn a seed, idea, or existing PRD into the right-sized plan while preserving schema gates. Planning only; never implements code."
---

# Shape to Plan

Produce the lightest correct planning artifact for the work:

```text
seed / idea / PRD -> shape-to-plan -> work/plan.md OR work/idea.md + work/PRD.md + work/kanban.md
```

This is a user-level harness over the planning semantics of `seed-to-idea`, `idea-to-prd`, and `prd-to-kanban`. Do not treat those skills as runtime functions. Reuse their schemas, source annotation rules, clarification gate behavior, PRD No-New-Fields rule, and Kanban pre-flight checks.

Do not implement code, spawn agents, or mark tasks complete.

## Inputs

Load the best available source:

- Raw prompt or pasted seed
- Seed file such as `seed.md`
- Existing `work/idea.md`
- Existing `work/PRD.md`
- Existing `work/praxiskit-context.md`

If multiple upstream artifacts exist, resume from the latest valid stage instead of restarting. Prefer `work/PRD.md` over `work/idea.md`, and `work/idea.md` over a raw seed.

## Required References

Read these before writing artifacts:

- `references/field-state-semantics.md`
- `references/clarification-gate.md`
- `schemas/idea.schema.md`
- `schemas/prd.schema.md`
- `schemas/kanban.schema.md`
- `schemas/plan.schema.md`

## Workflow

1. **Load context.** Read source artifacts and `work/praxiskit-context.md` if present.
2. **Classify mode.** Choose Light, Standard, or Full using the hard gates below.
3. **Map source to schemas.** Build one consolidated gap list across visible idea, PRD, kanban, and plan fields.
4. **Apply schema safety.**
   - `filled-by-user` fields require a user source.
   - `forbidden-to-infer` fields require a user source.
   - `inferable` fields must cite the user-sourced field they derive from.
   - The PRD No-New-Fields rule still applies in Standard/Full mode.
5. **Apply consolidated clarification gate.**
   - 0 gaps -> write artifacts directly.
   - 1-2 shallow gaps (`choice` or `short_text`) -> ask one question at a time.
   - 3+ gaps, or any `list`, `free_text`, or `multi_field` gap -> write `work/clarify-plan.md` and stop.
6. **Write artifacts for the chosen mode.**
   - Light -> `work/plan.md` + `work/praxiskit-context.md`
   - Standard -> `work/idea.md`, `work/PRD.md`, compact `work/kanban.md`, `work/praxiskit-context.md`
   - Full -> `work/idea.md`, `work/PRD.md`, `work/kanban.md`, `work/SUBAGENT.md`, `work/praxiskit-context.md`
7. **Stop before implementation.** Report generated paths and the recommended next skill.

## Mode Classifier

Choose the lightest mode that preserves correctness.

### Light Mode

Use Light only when all are true:

- One primary goal
- One acceptance surface
- Expected change is 1-3 files
- No delegation expected
- No public contract/API/schema change
- No external integration behavior
- No unresolved blocking or deep clarification gaps
- Expected implementation fits one local execution pass

Light mode writes `work/plan.md` governed by `schemas/plan.schema.md`. It may not invent domain lists, API details, schema fields, performance targets, security requirements, or integration requirements.

### Standard Mode

Use Standard when the work has multiple requirements or unclear scope but likely remains single-agent:

- Multiple functional requirements
- Product scope needs PRD structure
- Repo contract, package/plugin manifest, public API, schema, database migration, or integration behavior is touched
- Light mode eligibility is uncertain

Standard writes `work/idea.md`, `work/PRD.md`, and a compact `work/kanban.md`. Write `work/SUBAGENT.md` only if execution may need delegation.

### Full Mode

Use Full when the work likely needs multi-wave planning or delegation:

- Multiple independent write scopes
- Cross-module behavior
- Multi-wave dependencies
- Parallel workers would help
- High ambiguity that affects task decomposition

Full writes the complete PraxisKit artifact set, including `work/SUBAGENT.md`.

## Consolidated Clarification

Prefer one `work/clarify-plan.md` over separate stage-specific clarification files when gaps are visible from the source.

Use this path when writing a table:

```text
work/clarify-plan.md
```

Use the template from `references/clarification-gate.md`, with `stage = plan`. After writing it, stop and say:

> I need your input before I can write the plan. I created `work/clarify-plan.md` with the questions. Fill in the `<TBD: ...>` placeholders and say **continue** when done.

On `continue`, re-read `work/clarify-plan.md`, reject remaining `<TBD:` placeholders, archive the completed file to `work/clarify-archive/plan-{YYYY-MM-DD}-{HHMM}.md`, and then write the chosen artifacts with `[user via clarify-plan]` annotations.

## `work/plan.md` Format

Use this only for Light mode:

```markdown
# Plan: {name}

## Intent
{1-3 sentences with source annotations}

## Scope
- In: {included behavior/files} [{source}]
- Out: {excluded behavior/files or "none"} [{source}]

## Tasks
| ID | Task | Status | Acceptance Criteria | Write Scope | Dependencies | Source |
|---|---|---|---|---|---|---|
| L1 | ... | [ ] | Given..., when..., then... | `path` | [] | [inferred from scope_in] |

## Validation
- Build: `{command or unknown}` [{source}]
- Test: `{command or unknown}` [{source}]

## Review Handoff
`plan-to-review` should execute this plan and write `work/review.md`.
```

## Context Update

Create or update `work/praxiskit-context.md` and keep it under 40 lines:

```markdown
# PraxisKit Context: {project}

## Source Files
- plan: `work/plan.md` OR
- idea: `work/idea.md`
- prd: `work/PRD.md`
- kanban: `work/kanban.md`
- subagent: `work/SUBAGENT.md`

## Current Milestone
{mode and status}

## Canonical Constraints
- {schema, contract, API, or "none"}

## Open Blockers
- {gate or blocking question, or "none"}

## Latest Validation
- Planning only; no code validation run

## Last Updated By
shape-to-plan on {date}
```

## Handoff

End with one of:

- Light: "`work/plan.md` is ready. Use `plan-to-review` when you want execution; it dry-runs unless you explicitly authorize implementation."
- Standard/Full: "`work/kanban.md` is ready. Use `plan-to-review` or `kanban-to-agents` when you want execution."
