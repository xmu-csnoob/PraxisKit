# AGENTS.md

This repository packages `prd-to-kanban` for both Claude Code and Codex.

## Purpose

`prd-to-kanban` converts PRDs and feature requirements into:

- `work/kanban.md` - dynamic, agent-editable Kanban board
- `work/SUBAGENT.md` - static shared context for subagents

It is planning-only. Do not implement tasks as part of the skill workflow.

## Compatibility Surfaces

- Standalone skill: root `SKILL.md`
- Claude Code plugin: `.claude-plugin/plugin.json` plus `skills/prd-to-kanban/SKILL.md`
- Claude Code marketplace: `.claude-plugin/marketplace.json`
- Codex plugin: `.codex-plugin/plugin.json` plus `skills/prd-to-kanban/SKILL.md`
- Codex marketplace: `.agents/plugins/marketplace.json`

Root `SKILL.md` is the source of truth. Keep `skills/prd-to-kanban/SKILL.md` as a thin plugin entry that points back to root `SKILL.md`.
