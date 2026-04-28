# PRD to Kanban

[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code%20Marketplace-Listed-green?style=flat-square&logo=anthropic)](https://claude.com/claude-code/marketplace)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](#license)

> **Official Claude Code Marketplace Plugin** — Published to `xmu-csnoob-tools` plugin package. Install directly via `claude plugin install prd-to-kanban@xmu-csnoob-tools`

A cross-compatible Claude Code and Codex skill/plugin that converts PRDs or requirements into a living Kanban board (`work/kanban.md`) plus shared subagent context (`work/SUBAGENT.md`).

## Workflow

```
PRD → prd-to-kanban → kanban.md → subagent-driven-development → implemented code
```

This is a **planning-only** skill: it decomposes work but does not implement tasks. The generated files are ready for `subagent-driven-development` or equivalent multi-agent execution.

## Demo

Paste a PRD → skill generates `work/kanban.md` → agents execute in parallel layers → Done.

![PRD to Kanban Demo](docs/demo.gif)

> **Recording tip:** Use [Kap](https://getkap.co) or [LICEcap](http://www.cockos.com/licecap) to capture your screen. Save as GIF and place in `docs/demo.gif`.

## What the output looks like

Given a PRD, `work/kanban.md` produces a living board with topological layers and critical path markers:

````markdown
# kanban.md — Project Name

**Overall Progress:** 3/18 tasks (17%) | **Critical Path Length:** 11 layers
**Last Updated:** 2026-04-28 09:15 | **Orchestrator:** @agent-orchestrator

---

## 🔴 Critical Path (longest dependency chain)

| Task | Layer | Type | Status |
|------|-------|------|--------|
| Design database schema | 0 | schema | 🔴 IN PROGRESS |
| Implement auth service | 1 | API | 🔴 IN PROGRESS |
| Build login endpoint | 2 | API | ⬜ TODO |
| Create session middleware | 3 | feature | ⬜ TODO |
| Write auth integration tests | 5 | test | ⬜ TODO |

---

## 🟡 Layer 0 — Schema / Contract (parallelism: 3 tasks)

| Task | Type | Status | Blocked By |
|------|------|--------|------------|
| Design database schema | schema | 🔴 IN PROGRESS | — |
| Define API response contracts | schema | 🟡 IN PROGRESS | — |
| Design data models | schema | ⬜ TODO | — |

## 🟡 Layer 1 — Foundation APIs (parallelism: 4 tasks, depends on Layer 0)

| Task | Type | Status | Blocked By |
|------|------|--------|------------|
| Implement auth service | API | 🔴 IN PROGRESS | Design database schema |
| Build user CRUD endpoints | API | ⬜ TODO | Design database schema |
| Create project endpoints | API | ⬜ TODO | Design database schema |
| Define search interface | API | ⬜ TODO | Define API response contracts |

## 🟢 Layer 2 — Feature Logic (parallelism: 5 tasks, depends on Layer 1)

...

## Features

- **Topological task layering** — same layer = no transitive dependencies = safe to parallelize
- **Critical path identification** — longest chain through dependency graph, marked with visual indicators
- **Computed fields** — progress %, dependency layers, parallelism windows auto-refreshed from task statuses
- **Multi-agent update protocol** — orchestrator-owned board, subagent-safe status updates
- **Definition of Done** — verifiable acceptance criteria by task type (schema, API, UI, feature, test, integration)
- **Claude auto-install** — `/prd-to-kanban install` writes trigger rules to `CLAUDE.md`

## Layout

```text
SKILL.md                                      # Standalone Claude/Codex skill entry
plugins/prd-to-kanban/SKILL.md               # Shared skill source of truth
plugins/prd-to-kanban/skills/prd-to-kanban/  # Plugin-packaged skill entry
plugins/prd-to-kanban/.claude-plugin/        # Claude Code plugin manifest
plugins/prd-to-kanban/.codex-plugin/         # Codex plugin manifest
.claude-plugin/marketplace.json              # Claude Code marketplace catalog
.agents/plugins/marketplace.json             # Codex marketplace catalog
```

## Install

### Claude Code standalone skill

```bash
git clone https://github.com/xmu-csnoob/prd-to-kanban.git ~/.claude/skills/prd-to-kanban
```

Then run in Claude Code:

```text
/prd-to-kanban install
```

### Claude Code plugin

For local development:

```bash
claude --plugin-dir plugins/prd-to-kanban
```

For marketplace installation after publishing this repo:

```bash
claude plugin marketplace add xmu-csnoob/prd-to-kanban
claude plugin install prd-to-kanban@xmu-csnoob-tools
```

### Codex standalone skill

Clone or copy this repository to your Codex skills directory:

```bash
git clone https://github.com/xmu-csnoob/prd-to-kanban.git ~/.codex/skills/prd-to-kanban
```

Restart Codex so the skill list is refreshed.

### Codex plugin

For local testing from this repository:

```bash
codex plugin marketplace add .
```

For installation after publishing this repo:

```bash
codex plugin marketplace add xmu-csnoob/prd-to-kanban
```

Then open `/plugins`, choose `xmu-csnoob Tools`, and install `prd-to-kanban`.

## Usage

```text
/prd-to-kanban                         # Claude standalone skill
/prd-to-kanban install                 # Claude trigger rules in ~/.claude/CLAUDE.md
/prd-to-kanban install --project       # Claude trigger rules in ./CLAUDE.md
/prd-to-kanban:prd-to-kanban           # Claude plugin skill
Use $prd-to-kanban to plan this PRD.   # Codex skill/plugin invocation style
```

## Output

- `<project>/work/kanban.md`
- `<project>/work/SUBAGENT.md`

## License

MIT
