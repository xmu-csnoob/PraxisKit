# PraxisKit

**Turn a rough idea into shipped code — through governed handoffs between AI agents.**

PraxisKit is a Claude Code / Codex plugin that chains five skills into a full intent-to-acceptance pipeline. Each skill stops at a clean handoff boundary, preserving context as ideas become concrete.

```text
seed-to-idea    →  idea-to-prd  →  prd-to-kanban  →  kanban-to-agents  →  build-to-review
   ↓                  ↓               ↓                   ↓                   ↓
 work/idea.md    work/PRD.md    work/kanban.md      code changes         work/review.md
                                     + SUBAGENT.md
```

> Stop at any step. The output of each skill is the input to the next.

[![Claude Code Marketplace](https://img.shields.io/badge/Claude%20Code%20Marketplace-Published-green?style=flat-square&logo=anthropic)](https://github.com/xmu-csnoob/praxiskit/blob/main/.claude-plugin/marketplace.json)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](#license)
[![Stars](https://img.shields.io/github/stars/xmu-csnoob/praxiskit?style=flat-square&logo=github)](https://github.com/xmu-csnoob/praxiskit/stargazers)

## The Five Skills

| Skill | Use when... | Stops before |
|-------|-------------|--------------|
| `/praxiskit:seed-to-idea` | You have a rough thought, not a spec | Writing requirements |
| `/praxiskit:idea-to-prd` | You need structured requirements | Task breakdown |
| `/praxiskit:prd-to-kanban` | You have a PRD and need an execution plan | Implementation |
| `/praxiskit:kanban-to-agents` | You want coordinated multi-agent build | User acceptance |
| `/praxiskit:build-to-review` | Implementation is done — time to judge it | New scope |

## Quick Start

```bash
# 1. Add marketplace
claude plugin marketplace add xmu-csnoob/praxiskit

# 2. Install the plugin
claude plugin install praxiskit@xmu-csnoob-tools

# 3. Run — start anywhere in the chain
/praxiskit:seed-to-idea   # give it a rough idea
/praxiskit:idea-to-prd     # turn the idea into a PRD
/praxiskit:prd-to-kanban  # turn the PRD into a kanban board
```

For Codex: `codex plugin marketplace add xmu-csnoob/praxiskit`, then open `/plugins` → install `praxiskit`.

## What Gets Produced

### work/kanban.md — the living board

A dependency-aware task board with computed layers, parallelism windows, and a critical path. Agents edit it directly; you read the summary.

```
## Dependencies & Parallelism
### Dependency Layers (topological order)
| Layer | Tasks    | Depends On | Wave |
|-------|----------|------------|------|
| L0    | T0.1     | —          | W0   |
| L1    | T0.2     | L0         | W0   |
| L2    | T1.1,T1.2| L0..L1     | W1   |

### Critical Path
T0.1 → T0.2 → T1.1
```

### work/SUBAGENT.md — shared agent context

Every subagent reads this before touching anything. Stack, frozen contracts, write scopes, reporting convention — all in one place, never inferred.

## Install

### Claude Code plugin (recommended)

```bash
claude plugin marketplace add xmu-csnoob/praxiskit
claude plugin install praxiskit@xmu-csnoob-tools
```

### Claude Code — local dev from this repo

```bash
claude --plugin-dir plugins/praxiskit
```

### Codex plugin

```bash
codex plugin marketplace add xmu-csnoob/praxiskit
# then open /plugins → xmu-csnoob Tools → install praxiskit
```

### Standalone skill (no plugin)

```bash
git clone https://github.com/xmu-csnoob/praxiskit.git ~/.claude/skills/praxiskit
```

## Calling Individual Skills

All five skills live in the same plugin. Use `:` to invoke a specific step:

```text
/praxiskit:seed-to-idea        /praxiskit:idea-to-prd
/praxiskit:prd-to-kanban       /praxiskit:kanban-to-agents
/praxiskit:build-to-review
```

No need to install each skill separately — one plugin install gives you all five.

## Why PraxisKit

Most agent frameworks hand off via chat history. PraxisKit hands off via **artifacts**: structured documents that each skill writes and the next skill reads. The chain forces intent to become concrete before work begins, and forces review before work is accepted.

- **Idea** → vague intent is shaped into a document
- **PRD** → intent is translated into explicit requirements
- **Kanban** → requirements are decomposed into dependency-ordered tasks
- **Agents** → tasks are executed with explicit write scopes
- **Review** → output is validated against the original idea

## Examples

See `examples/` in this repo:
- `sample-prd.md` — example PRD input
- `sample-kanban.md` — example kanban board output
- `sample-praxiskit-context.md` — example SUBAGENT.md

## Repository Layout

```
plugins/praxiskit/skills/
  seed-to-idea/         # raw seed → work/idea.md
  idea-to-prd/          # idea → work/PRD.md
  prd-to-kanban/        # PRD → work/kanban.md + work/SUBAGENT.md
  kanban-to-agents/     # kanban → coordinated implementation
  build-to-review/      # build → acceptance packet
```

## License

MIT
