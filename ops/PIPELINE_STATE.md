# Pipeline State

Current state of the build pipeline. Updated after every significant action.

## Last Updated: 2026-03-14 17:30

## Active Work

| Item | Status | Branch | PR | Blockers |
|------|--------|--------|----|----------|
| PR #166: D2 dashboard + auth + cost + shell | Draft, reviewed | `codex/yolo-build-fix` | [#166](https://github.com/NYQST-Group/NYQST-DocuIntelli-Build/pull/166) | 2 critical issues from review |
| PR #165: Backstage catalog refresh | Draft, stale (Mar 11) | `copilot/sub-pr-164` | [#165](https://github.com/NYQST-Group/NYQST-DocuIntelli-Build/pull/165) | Needs triage |
| PR #164: Backstage extraction | Non-draft | `chore/extract-backstage` | [#164](https://github.com/NYQST-Group/NYQST-DocuIntelli-Build/pull/164) | Needs review |
| YOLO PR #37: D2-01 contract lock | Draft | `feat/d2-01-contract-lock-*` | [#37](https://github.com/NYQST-Group/NYQST-DocuIntelli-Execution-YOLO/pull/37) | Needs review |
| YOLO PR #38: D3-01 console contract | Draft | `feat/d3-01-console-contract-*` | [#38](https://github.com/NYQST-Group/NYQST-DocuIntelli-Execution-YOLO/pull/38) | Ahead of schedule |

## Pending Review Issues (from PR #166)

| ID | Severity | Issue | File | Status |
|----|----------|-------|------|--------|
| C1 | Critical | `format_cost_usd` edge case | `pricing.py:64-66` | OPEN |
| C2 | Critical | Raw SQL in `get_cost_breakdown` | `session_service.py:150-156` | OPEN |
| I1 | Important | Zero tests for shell API (427 lines) | `shell.py` | OPEN |
| I2 | Important | Duplicate `formatCost` in 3 files | frontend | OPEN |
| I3 | Important | Duplicate token aggregation in 4 places | backend+frontend | OPEN |
| I4 | Important | Unknown models → $0 cost silently | `pricing.py` | OPEN |
| I7 | Important | LeftRail logout missing navigate | `LeftRail.tsx:103` | OPEN |

## Project 11 Delivery Status

| Deliverable | Board Status | Actual Status |
|-------------|-------------|---------------|
| D1 Shell, identity, context | Done | Done (5/5 closed) |
| D2 Dashboard, projects, clients | In Progress | PR #166 has partial work, YOLO PR #37 has D2-01 |
| D2-01 Lock contracts | Ready | YOLO PR #37 exists (draft) |
| D2-02 Persistence + endpoints | Inbox | Not started |
| D2-03 Real buyer surfaces | Inbox | Partial (OverviewPage in PR #166) |
| D3–D6 | Inbox | D3-01 has YOLO PR #38 (draft, ahead of schedule) |
| G1 Demo guardrails | Inbox | Partial (reporting infra in PR #166) |

## Uncommitted State
- Build repo: `CLAUDE.md` modified (test docs update) — needs committing

## Next Actions (Priority Order)

1. **Dispatch coding agent** to fix C1 + C2 critical issues on PR #166
2. **Dispatch reviewer agent** to verify the fixes
3. **Dispatch test analyzer** to assess shell API test gap (I1)
4. **If all clear**, merge PR #166
5. **Triage YOLO PR #37** (D2-01) — review and advance
6. **Triage stale PR #165** — close or update
7. Start D2-02: land thin project/client persistence + dashboard endpoints
