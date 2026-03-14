# Agent Ledger

Tracks every agent type, their capabilities, performance, and promotion status.

## Agent Registry

| Agent ID | Type | Platform | Strengths | Weaknesses | Trust Level | Notes |
|----------|------|----------|-----------|------------|-------------|-------|
| — | — | — | — | — | — | No agents tested yet |

### Trust Levels
- `untested` — never used, try on small tasks first
- `probation` — used once, needs more data
- `trusted` — consistent good results on moderate tasks
- `promoted` — proven on hard tasks, preferred for that domain

## Agent Types Available

### Claude Code Subagents (via Agent tool)
| subagent_type | Tools | Best for | Trust |
|---------------|-------|----------|-------|
| `general-purpose` | All | Coding, multi-step tasks | untested |
| `Explore` | Read-only | Codebase research | untested |
| `Plan` | Read-only | Architecture planning | untested |
| `feature-dev:code-reviewer` | Read-only | Code quality review | untested |
| `feature-dev:code-explorer` | Read-only | Deep codebase analysis | untested |
| `feature-dev:code-architect` | Read-only | Architecture design | untested |
| `pr-review-toolkit:review-pr` | All | Comprehensive PR review | untested |
| `pr-review-toolkit:silent-failure-hunter` | All | Error handling gaps | untested |
| `pr-review-toolkit:pr-test-analyzer` | All | Test coverage analysis | untested |
| `pr-review-toolkit:code-reviewer` | All | Guidelines adherence | untested |
| `pr-review-toolkit:code-simplifier` | All | Code simplification | untested |
| `pr-review-toolkit:type-design-analyzer` | All | Type design quality | untested |
| `superpowers:code-reviewer` | All | Production readiness review | probation |
| `code-simplifier:code-simplifier` | All | Simplification | untested |

### External Agents
| Agent | Platform | Best for | Trust |
|-------|----------|----------|-------|
| Codex CLI (`o4-mini`) | Terminal | Fast focused coding | untested |
| Codex CLI (`o3`) | Terminal | Complex coding | untested |
| Jules | GitHub Cloud | Long-running autonomous tasks | untested |

## Promotion History

| Date | Agent | From | To | Reason |
|------|-------|------|----|--------|
| 2026-03-14 | `superpowers:code-reviewer` | untested | probation | Used on PR #166, produced thorough 7-issue review with correct severity categorization |
| 2026-03-14 | `Explore` | untested | probation | Ingested entire project docs (11 PRDs, 10 ADRs, build plan) in 82s with structured summary |
| 2026-03-14 | `general-purpose` | untested | probation | Two successful runs: issue inventory (101s) and agent tooling check (43s), both accurate and actionable |
| 2026-03-14 | `general-purpose (haiku)` | untested | probation | Capability probe: confirmed Skill tool access in subagents, accurate and fast (12s) |
| 2026-03-14 | `general-purpose (sonnet)` | untested | probation | TDD test: followed full RED-GREEN-REFACTOR on format_cost_usd, loaded skill, honest reporting (49s) |
