# Loop Architecture

## The Problem with "Finish A Before Starting B"

Too restrictive. In a real pipeline, phases can overlap across work items:
- SPEC for D2-02 can run while D2-01 is being IMPLEMENTED (spec doesn't touch code)
- REVIEW prep for D2-01 can run while D2-01 is still being IMPLEMENTED
- But IMPLEMENTATION of D2-02 CANNOT start until D2-01's contracts are merged (code dependency)

This is pipelined execution, not sequential execution.

## Loop Hierarchy

```
OUTER LOOP: Ship demo-ready product
  Outcome: All 35 Project 11 items Done
  Exit: All waves complete OR blocked on user decision
  Visibility: Project 11 board progress

  WAVE LOOP: Complete delivery wave (D2, D3, etc.)
    Outcome: All PKTs merged + WRAP done for this wave
    Exit: WRAP merged to main → advance to next wave
    Visibility: PKT completion count within wave

    PKT LOOP: Deliver one work packet
      Outcome: PR merged to main, issue closed
      Exit: Merge confirmed + board updated
      Visibility: Current phase (SPEC/IMPLEMENT/VERIFY/REVIEW/FIX/MERGE)

      IMPLEMENTATION LOOP: Single task within a PKT
        Outcome: Tests pass, TDD followed, code committed
        Exit: Doer reports DONE + checker validates
        Visibility: TDD cycle (RED/GREEN/REFACTOR)

      REVIEW LOOP: Iterate until quality gate passes
        Outcome: All Critical+Important issues resolved
        Exit: Reviewer approves + meta-reviewer confirms
        Visibility: Issue count (Critical/Important/Minor remaining)
```

## What Can Overlap (Pipelining Rules)

### Within a Wave (e.g., D2 has PKTs D2-01, D2-02, D2-03, D2-WRAP)

```
TIME →
D2-01:  [SPEC]──[IMPLEMENT]──[VERIFY]──[REVIEW]──[FIX]──[MERGE]
D2-02:       [SPEC]─────────────────[wait for D2-01 merge]──[IMPLEMENT]──...
D2-03:            [SPEC]────────────────────────────────[wait for D2-02]──...
D2-WRAP:                                                              [after all]
```

| Phase for PKT N+1 | Can overlap with PKT N in... | Why |
|---|---|---|
| SPEC | Any phase | Spec is read-only: reads issues, explores codebase, writes plan |
| IMPLEMENT | Only after N is MERGED | Code depends on N's contracts being in main |
| VERIFY | Never overlaps | Each PKT verifies its own work |
| REVIEW | Only after N is MERGED | Reviews N+1's diff which needs N's code |
| MERGE | Never overlaps | Strictly sequential to main |

**The big win:** Front-load ALL specs in a wave. While D2-01 is being implemented,
D2-02 and D2-03 specs are already written and reviewed. When D2-01 merges,
D2-02 implementation starts INSTANTLY.

### Across Waves (D2, D3, D4, etc.)

Normally strictly sequential. But:
- G1 (guardrails) items that don't depend on specific surfaces CAN interleave
- D3-01 (contract lock) is already drafted in YOLO PR #38 — acceptable if contract-only

### Within a PKT (implementation tasks)

| Can overlap? | Rule |
|---|---|
| Two implementation tasks on same branch | NO — commit conflicts |
| Implementation + its own spec review | NO — sequential |
| Implementation + prep-ahead for next task | YES — prep is read-only |
| Two reviewers on same PR | YES — reviews are read-only |
| Fix + re-review | NO — sequential |

## WIP Limits

| Stage | Max WIP | Why |
|---|---|---|
| SPEC | 3 | Can prep multiple PKTs ahead — read-only, no conflicts |
| IMPLEMENT | 1 | Only one coding agent on one branch at a time |
| VERIFY | 1 | Verify the thing just implemented |
| REVIEW | 3 | Multiple reviewers in parallel on same PR — read-only |
| FIX | 1 | One fix agent per branch |
| MERGE | 1 | Sequential to main |

## Visibility Dashboard

The orchestrator should maintain this view in ops/PIPELINE_STATE.md:

```
═══ DEMO-READY: 5/35 items ═══════════════════════════ ETA: ~20h agent time

WAVE D2: 1/5 done ──────────────────────────────────── ETA: ~3h
  PR #166 merge    │ FIXING   │ C1[pending] → C2[pending] → verify → review → merge
  D2-01 #7         │ SPEC OK  │ YOLO PR #37 exists → REVIEW IT (don't duplicate)
  D2-02 #8         │ SPEC OK  │ Can front-load spec now. IMPL blocked on PR #166 merge.
  D2-03 #9         │ CAN SPEC │ Can front-load spec now. IMPL blocked on D2-02.
  D2-WRAP #10      │ WAITING  │ After all PKTs.

MONITORS: 3 crons active (health :07, proj11 :03, ci :11)
AGENTS IN FLIGHT: 0
PREP QUEUE: D2-02 spec, D2-03 spec (can start now)
```

## Outcome-Based Loop Exits

Each loop has a clear exit condition — not "tasks done" but "outcome achieved":

| Loop | Exit Condition | How to Verify |
|---|---|---|
| Outer | All Project 11 items Done | `gh project item-list 11` — 35/35 Done |
| Wave | All PKTs + WRAP merged for this wave | 5/5 items Done in wave |
| PKT | PR merged to main, GitHub issue closed | `gh pr view --json state` = MERGED + issue closed |
| Implementation | Tests pass, TDD followed, code committed | Test output shows 0 failures + TDD checker approves |
| Review | All Critical+Important resolved, reviewer approves | Review report shows 0 Critical, 0 Important open |

## Pause and Resume

**Pause (between outcomes):**
Best pause points are between PKTs — a clean state where:
- Current PKT is merged
- Next PKT spec is ready
- ops/ committed and pushed

**Pause (mid-PKT):**
Acceptable but messier. Record in PIPELINE_STATE.md:
- Which implementation task was last completed
- Which task is next
- Worktree location and branch name
- What's been committed vs uncommitted

**Resume:**
Read PIPELINE_STATE.md → detect pause state → re-dispatch from last checkpoint.

## Recovery Scenarios

| Scenario | Detection | Recovery |
|---|---|---|
| Session dies mid-implementation | TaskList shows `in_progress` task | Check worktree git log → resume or re-dispatch |
| Agent fails repeatedly | RUN_LOG shows 2+ failures | Escalate to more capable model or different approach |
| Tests break after merge | CI monitor cron reports failure | Revert or hotfix on main (create urgent issue) |
| Board drifts from ops state | Proj11 sync cron reports mismatch | Update PIPELINE_STATE.md from live board |
| Conflicting PRs | CI/PR monitor reports conflicts | Rebase the newer PR |
