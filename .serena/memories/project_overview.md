# NYQST-YOLOYOLO: Build Pipeline Orchestrator

## Purpose
This is NOT a code repository. It is the **build pipeline management workspace** for the NYQST DocuIntelli platform. The operator (Claude) manages a multi-agent development pipeline from here, dispatching coding/review/test work to agents and tracking progress.

## What This Repo Contains
- `CLAUDE.md` — Operating manual for the build pipeline manager
- `ops/PIPELINE_STATE.md` — Current pipeline state, blockers, next actions
- `ops/RUN_LOG.md` — Append-only chronological log of agent dispatches
- `ops/AGENT_LEDGER.md` — Agent registry with capabilities and trust levels
- `.claude/hookify.*.local.md` — Enforcement hooks preventing direct coding

## Managed Repos
| Repo | Path | Purpose |
|------|------|---------|
| Build | ~/NYQST-DocuIntelli-Build | Production codebase (Python + React) |
| Execution | ~/NYQST-DocuIntelli-Execution | Execution environment |
| Execution-YOLO | ~/NYQST-DocuIntelli-Execution-YOLO | Fast-iteration fork |

## GitHub Project 11
- Tracks 35 deliverable items (D1–D6 + G1)
- CAP → PKT → WRAP hierarchy per deliverable
- CLI: `gh project item-list 11 --owner NYQST-Group --format json`

## Issue Landscape
- Build repo: 159 open issues across 23 milestones, 0 closed
- YOLO repo: 30 open issues (all demo-critical), 5 closed
- 125 requirements in NYQST registry, 9 value drivers
