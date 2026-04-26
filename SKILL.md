---
name: prd-to-kanban
description: "Convert a PRD, design doc, or feature requirements into work/kanban.md and work/SUBAGENT.md for multi-agent implementation planning. Planning only. Sub-command: install for Claude Code trigger rules."
---

# PRD to Kanban

This is the standalone skill entry for `prd-to-kanban`.

Read the shared implementation instructions at `plugins/prd-to-kanban/SKILL.md` and follow them exactly. That file is the source of truth for triggers, output format, task granularity, acceptance criteria, dependency layering, and handoff rules.

If `plugins/prd-to-kanban/SKILL.md` is unavailable, use this fallback:
- Convert the PRD or requirements into two files: `<project>/work/kanban.md` and `<project>/work/SUBAGENT.md`.
- Plan only. Do not implement tasks.
- Use verifiable acceptance criteria for every task.
- Put frozen contract/schema work in Wave 0.
- Compute dependency layers, parallelism windows, and the critical path.
- Keep `SUBAGENT.md` under 80 lines with project summary, stack, frozen contracts, write scopes, and subagent reporting conventions.
- Report the generated paths. Do not open files or launch external viewers unless the user explicitly asks.
