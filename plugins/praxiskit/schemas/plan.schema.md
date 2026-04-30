---
schema_name: plan
schema_version: "2.1"
used_by: shape-to-plan, plan-to-review
output_file: work/plan.md
---

# Plan Schema v2.1

Defines the compact `work/plan.md` artifact used by Light mode. This schema does not bypass PraxisKit v2 safety rules: every value must be user-sourced, collected through the clarification gate, or inferable from a user-sourced field.

## Required Fields

| Field | State | Kind | Notes |
|---|---|---|---|
| name | inferable | short_text | Infer from seed, explicit request, or repo context |
| intent | filled-by-user or inferable | short_text | Must trace to seed or gate answer |
| scope_in | filled-by-user | list | Included behavior/files. List fields require user source unless explicitly present in seed |
| scope_out | filled-by-user | list | Can be empty only when user confirms none |
| tasks | filled-by-user or inferable | list | Derive only from confirmed intent and scope |
| task.acceptance_criteria | inferable | short_text | Must be observable Given/When/Then; user can override |
| task.write_scope | inferable | list | Derive from repo layout and task target |
| task.dependencies | inferable | list | Derive from task ordering and write-scope constraints |
| validation | inferable | list | Derive from repo scripts; `unknown` allowed if no scripts exist |

## Forbidden-to-Infer Fields

These fields MUST NOT appear in a compact plan unless the user provided them directly or through `work/clarify-plan.md`:

| Field | Kind | Gate Question |
|---|---|---|
| domain_specific_lists | list | "What exact domain items does this plan need?" |
| performance_targets | list | "What performance targets apply? (or 'none for now')" |
| public_api_details | list | "What public API details are required? List endpoints, commands, flags, or exported names." |
| schema_or_contract_fields | list | "What schema or contract fields must be included?" |
| security_or_compliance_requirements | list | "What security, privacy, or compliance requirements apply? (or 'none')" |
| external_integrations | list | "Which external systems must this integrate with? (or 'none')" |

## Light Mode Eligibility

Light mode may emit `work/plan.md` only when all are true:

- One primary goal
- One acceptance surface
- No delegation expected
- No public contract/API/schema change
- No unresolved blocking or deep clarification gaps
- Expected implementation fits one local execution pass

If any eligibility item is uncertain, `shape-to-plan` must choose Standard or Full mode.

## Output Format

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

## Source Annotation Rules

Source values: `[user]`, `[user via gate]`, `[user via clarify-plan]`, `[inferred from {field}]`, `[default]`.

## Changelog

- v2.1 (2026-04-29): Initial compact plan schema for integrated `shape-to-plan` Light mode.
