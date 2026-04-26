# AGENTS.md

This repository packages `prd-to-kanban` for both Claude Code and Codex.

## Purpose

`prd-to-kanban` converts PRDs and feature requirements into:

- `work/kanban.md` - dynamic, agent-editable Kanban board
- `work/SUBAGENT.md` - static shared context for subagents

It is planning-only. Do not implement tasks as part of the skill workflow.

## Compatibility Surfaces

- Standalone skill: root `SKILL.md`
- Shared skill source: `plugins/prd-to-kanban/SKILL.md`
- Claude Code plugin: `plugins/prd-to-kanban/.claude-plugin/plugin.json` plus `plugins/prd-to-kanban/skills/prd-to-kanban/SKILL.md`
- Claude Code marketplace: `.claude-plugin/marketplace.json`
- Codex plugin: `plugins/prd-to-kanban/.codex-plugin/plugin.json` plus `plugins/prd-to-kanban/skills/prd-to-kanban/SKILL.md`
- Codex marketplace: `.agents/plugins/marketplace.json`

`plugins/prd-to-kanban/SKILL.md` is the source of truth. Keep root `SKILL.md` and `plugins/prd-to-kanban/skills/prd-to-kanban/SKILL.md` as thin entries.
