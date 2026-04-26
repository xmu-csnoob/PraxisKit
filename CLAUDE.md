# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A cross-compatible Claude Code and Codex skill/plugin that converts PRDs/requirements into `work/kanban.md` and `work/SUBAGENT.md`. This is a planning-only skill — it decomposes work but does not execute it. Execution is handed off to `subagent-driven-development`.

Workflow chain: PRD → **prd-to-kanban** → `kanban.md` → subagent-driven-development → implemented code.

Sub-commands:
- `install [--project]` — writes auto-trigger rules to CLAUDE.md so the skill fires automatically on PRD/requirements detection

## File Structure

```
SKILL.md                                      # Standalone skill entry
plugins/prd-to-kanban/SKILL.md               # Shared skill source of truth
plugins/prd-to-kanban/skills/prd-to-kanban/  # Plugin-packaged skill entry
plugins/prd-to-kanban/.claude-plugin/        # Claude Code plugin manifest
plugins/prd-to-kanban/.codex-plugin/         # Codex plugin manifest
.claude-plugin/marketplace.json              # Claude Code marketplace catalog
.agents/plugins/marketplace.json             # Codex marketplace catalog
CLAUDE.md                                     # This file - repo context for Claude Code
AGENTS.md                                     # Repo context for Codex
README.md                                     # User-facing install/usage docs
```

`plugins/prd-to-kanban/SKILL.md` is the single source of truth for the skill's behavior. Root `SKILL.md` and the plugin-packaged skill entry delegate to it.

## Modifying This Skill

- All skill logic is in `plugins/prd-to-kanban/SKILL.md` - edit that file directly
- Keep root `SKILL.md` and `plugins/prd-to-kanban/skills/prd-to-kanban/SKILL.md` as thin entries that point to the shared source
- The skill defines its own trigger conditions (when Claude should invoke it automatically)
- Settings in `.claude/settings.local.json` are local-only and gitignored. Keep them machine-specific.
- Test changes by running the skill against a sample PRD and inspecting the generated `kanban.md` and `SUBAGENT.md`

## Key Design Constraints

- **Two output files**: `kanban.md` (dynamic status tracker) + `SUBAGENT.md` (static shared context, under 80 lines). Orchestrator passes `SUBAGENT.md` to every subagent — subagents read it themselves.
- **Agents edit, humans read**: The board is a living document updated by orchestrating agents during execution
- **Topological layering drives parallelism**: Same layer + disjoint write scopes = safe to parallelize
- **Frozen interfaces first**: Schema/contract tasks are always Wave 0
