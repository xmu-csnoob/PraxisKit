# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A cross-compatible Claude Code and Codex skill/plugin that converts PRDs/requirements into `work/kanban.md` and `work/SUBAGENT.md`. This is a planning-only skill — it decomposes work but does not execute it. Execution is handed off to `subagent-driven-development`.

Workflow chain: PRD → **prd-to-kanban** → `kanban.md` → subagent-driven-development → implemented code.

Sub-commands:
- `install [--project]` — writes auto-trigger rules to CLAUDE.md so the skill fires automatically on PRD/requirements detection

## File Structure

```
SKILL.md                         # Standalone skill entry and source of truth
skills/prd-to-kanban/SKILL.md    # Plugin-packaged skill entry for Claude Code and Codex
CLAUDE.md                        # This file — repo context for Claude Code
README.md                        # User-facing install/usage docs
LICENSE                          # MIT
.claude/settings.local.json      # Local permissions (gitignored — configure per-machine)
.claude-plugin/plugin.json       # Claude Code plugin manifest
.claude-plugin/marketplace.json  # Claude Code marketplace catalog
.codex-plugin/plugin.json        # Codex plugin manifest
.agents/plugins/marketplace.json # Codex repo marketplace catalog
.gitignore                       # Ignores .DS_Store + settings.local.json
```

`SKILL.md` is the single source of truth for the skill's behavior. The plugin-packaged skill entry delegates back to it.

## Modifying This Skill

- All skill logic is in `SKILL.md` — edit that file directly
- Keep `skills/prd-to-kanban/SKILL.md` as a thin plugin entry that points back to root `SKILL.md`
- The skill defines its own trigger conditions (when Claude should invoke it automatically)
- Settings in `.claude/settings.local.json` are local-only and gitignored. Keep them machine-specific.
- Test changes by running the skill against a sample PRD and inspecting the generated `kanban.md` and `SUBAGENT.md`

## Key Design Constraints

- **Two output files**: `kanban.md` (dynamic status tracker) + `SUBAGENT.md` (static shared context, under 80 lines). Orchestrator passes `SUBAGENT.md` to every subagent — subagents read it themselves.
- **Agents edit, humans read**: The board is a living document updated by orchestrating agents during execution
- **Topological layering drives parallelism**: Same layer + disjoint write scopes = safe to parallelize
- **Frozen interfaces first**: Schema/contract tasks are always Wave 0
