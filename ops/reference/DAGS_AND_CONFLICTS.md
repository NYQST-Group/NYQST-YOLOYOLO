# DAGs, Timings, and Conflict Analysis

## DAG 1: PR #166 Critical Fix → Merge (IMMEDIATE)

```
                    ┌──────────────────┐     ┌──────────────────┐
                    │ T1: Fix C1 #167  │     │ T2: Fix C2 #168  │
                    │ format_cost_usd  │     │ move SQL to repo │
                    │ ~2 min (sonnet)  │     │ ~5 min (sonnet)  │
                    │ Branch: codex/   │     │ Branch: codex/   │
                    │ yolo-build-fix   │     │ yolo-build-fix   │
                    │ Files: pricing.py│     │ Files:           │
                    │                  │     │  session_service  │
                    │                  │     │  sessions.py repo │
                    └────────┬─────────┘     └────────┬─────────┘
                             │                         │
                             ▼                         ▼
                    ┌──────────────────────────────────────────┐
                    │ T3: Verify tests pass                    │
                    │ Run: pytest (506) + vitest (274) + tsc   │
                    │ ~2 min (bash commands, no agent needed)  │
                    │ BLOCKED BY: T1 AND T2                    │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ T4: Review fixes (sonnet code-reviewer)  │
                    │ ~3 min                                    │
                    │ BLOCKED BY: T3                            │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ T5: Merge PR #166                         │
                    │ gh pr ready + gh pr merge --squash        │
                    │ ~1 min                                    │
                    │ BLOCKED BY: T4                            │
                    └──────────────────────────────────────────┘

Total: ~13 min
Critical path: T2 (5 min) → T3 (2 min) → T4 (3 min) → T5 (1 min) = 11 min
```

### CONFLICTS in DAG 1:
| Conflict | Risk | Mitigation |
|----------|------|------------|
| **T1 and T2 edit same branch** | HIGH — both on `codex/yolo-build-fix`, both need to commit | Run T1 and T2 SEQUENTIALLY on same branch, OR use worktrees |
| **T1 and T2 edit same file** | LOW — pricing.py vs session_service.py/sessions.py — different files | No mitigation needed |
| **Cron fires during T1/T2** | LOW — cron reads only, doesn't write | No mitigation needed |
| **PR #166 gets external commit** | MEDIUM — if someone else pushes to the branch | Check `git fetch && git status` before starting |

**DECISION: T1 and T2 must be SEQUENTIAL on same branch, not parallel.**
Even though the files don't overlap, two agents committing to the same branch simultaneously
will cause push conflicts. Run T1 first, then T2 on the updated branch.

Revised timing: 2 + 5 + 2 + 3 + 1 = **13 min sequential**.

---

## DAG 2: D2-01 PKT Delivery (NEXT after PR #166)

```
                    ┌──────────────────────────────────────────┐
                    │ P0: Read issue #7 + acceptance criteria  │
                    │ ~1 min (you read it, cheap)              │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ P1: Explore codebase for existing        │
                    │     patterns (Explore agent, sonnet)     │
                    │ ~2 min background                        │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ P2: Write spec / plan (you or Plan agent)│
                    │ Save to ops/runs/7-spec.md               │
                    │ ~5 min                                   │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ P3: Spec review (sonnet reviewer)        │
                    │ ~2 min                                   │
                    │ Loop: fix → re-review (max 5 iterations) │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ P4: Create worktree from main            │
                    │ git worktree add /tmp/pkt-7-wt           │
                    │     -b feat/7-d2-01-contract main        │
                    │ ~10 sec                                  │
                    └────────────────────┬─────────────────────┘
                                         │
                    ┌────────────────────┼────────────────────┐
                    │                    │                     │
                    ▼                    ▼                     ▼
           ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
           │ I1: Task 1   │   │ I2: Task 2   │   │ I3: Task 3   │
           │ (sonnet,TDD) │   │ (sonnet,TDD) │   │ (sonnet,TDD) │
           │ ~5-10 min    │   │ ~5-10 min    │   │ ~5-10 min    │
           │ SEQUENTIAL   │   │ BLOCKED: I1  │   │ BLOCKED: I2  │
           └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
                  │                  │                    │
                  ▼                  ▼                    ▼
           ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
           │ R1: Spec     │   │ R2: Spec     │   │ R3: Spec     │
           │ review       │   │ review       │   │ review       │
           │ ~2 min       │   │ ~2 min       │   │ ~2 min       │
           └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
                  │                  │                    │
                  ▼                  ▼                    ▼
           ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
           │ Q1: Quality  │   │ Q2: Quality  │   │ Q3: Quality  │
           │ review       │   │ review       │   │ review       │
           │ ~2 min       │   │ ~2 min       │   │ ~2 min       │
           └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
                  │                  │                    │
                  └──────────────────┼────────────────────┘
                                     │
                                     ▼
                    ┌──────────────────────────────────────────┐
                    │ V1: Full test suite verification          │
                    │ pytest + vitest + tsc + ruff              │
                    │ ~3 min                                    │
                    │ BLOCKED BY: all I+R+Q tasks              │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ V2: Final code review (whole PKT diff)   │
                    │ sonnet code-reviewer                     │
                    │ ~3 min                                   │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ M1: Draft PR + push                      │
                    │ gh pr create --draft                     │
                    │ ~1 min                                   │
                    └────────────────────┬─────────────────────┘
                                         │
                    ┌────────────────────┼────────────────────┐
                    │                    │                     │
                    ▼                    ▼                     ▼
           ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
           │ PR1: code-   │   │ PR2: pr-     │   │ PR3: test-   │
           │ reviewer     │   │ review-pr    │   │ analyzer     │
           │ ~3 min       │   │ ~5 min       │   │ ~3 min       │
           │ PARALLEL     │   │ PARALLEL     │   │ PARALLEL     │
           └──────┬───────┘   └──────┬───────┘   └──────┬───────┘
                  │                  │                    │
                  └──────────────────┼────────────────────┘
                                     │
                                     ▼
                    ┌──────────────────────────────────────────┐
                    │ MR: Meta-review (sonnet — check reviews) │
                    │ ~2 min                                    │
                    │ BLOCKED BY: PR1 AND PR2 AND PR3          │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ FIX: Address review findings              │
                    │ ~5-15 min (depends on findings)           │
                    │ Loop: fix → re-verify → re-review        │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ MERGE: gh pr ready + merge               │
                    │ ~1 min                                    │
                    └────────────────────┬─────────────────────┘
                                         │
                                         ▼
                    ┌──────────────────────────────────────────┐
                    │ CLOSE: Update board, close issues (bg)   │
                    │ ~1 min (background sonnet)               │
                    └──────────────────────────────────────────┘

Total: 40-65 min per PKT (depending on review loops and task count)
Critical path: P0→P1→P2→P3→P4→I1→R1→Q1→I2→R2→Q2→I3→R3→Q3→V1→V2→M1→PR→MR→MERGE
```

### CONFLICTS in DAG 2:
| Conflict | Risk | Mitigation |
|----------|------|------------|
| **I1-I3 on same branch** | HIGH — sequential commits on same worktree branch | Tasks MUST be sequential (per subagent-driven-dev skill: "never dispatch parallel implementers") |
| **I1 edits file that I2 also needs** | MEDIUM — plan must assign non-overlapping file ownership | Spec must declare file ownership per task. If overlap detected, merge into one task. |
| **Prep-ahead agent reads while doer writes** | LOW — prep agent is read-only | No mitigation needed |
| **Cron fires during implementation** | LOW — cron reads only | No mitigation needed |
| **PR merge conflicts with main** | MEDIUM — main may have changed since worktree created | Rebase worktree branch before PR: `git rebase main` |
| **PR reviews overlap with fix cycle** | LOW — reviews are read-only, fixes are sequential | No mitigation needed |
| **Session death during I2** | HIGH — partially committed work in worktree | Recovery: next session reads worktree git log, resumes or re-dispatches from last commit |
| **Existing YOLO PR #37 (D2-01)** | HIGH — someone already created a D2-01 PR! | Must review PR #37 first. If it's good, review+merge it instead of creating new work. |

---

## DAG 3: D2 Full Wave (D2-01 → D2-02 → D2-03 → D2-WRAP)

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ D2-01    │     │ D2-02    │     │ D2-03    │     │ D2-WRAP  │
│ Contract │────▶│ Persist  │────▶│ Surfaces │────▶│ Polish   │
│ lock     │     │ + endpts │     │ buyer UI │     │ wrapper  │
│          │     │          │     │          │     │          │
│ #7       │     │ #8       │     │ #9       │     │ #10      │
│ 40-65min │     │ 40-65min │     │ 40-65min │     │ 30-45min │
└──────────┘     └──────────┘     └──────────┘     └──────────┘

TOTAL: 150-240 min (2.5-4 hours)
```

### CONFLICTS in DAG 3:
| Conflict | Risk | Mitigation |
|----------|------|------------|
| **D2-01 and D2-02 share schema files** | HIGH — D2-01 defines contracts that D2-02 implements | Strictly sequential: D2-01 must be merged to main before D2-02 branches |
| **D2-02 and D2-03 share API/UI surface** | HIGH — D2-02 creates endpoints that D2-03 consumes | Strictly sequential: D2-02 merged before D2-03 starts |
| **PR #166 already has partial D2 work** | HIGH — OverviewPage, session types already in PR #166 | D2-02/D2-03 must branch from main AFTER PR #166 merges |
| **YOLO PR #37 exists for D2-01** | HIGH — work may already be done | Review PR #37 first. Don't duplicate. |
| **YOLO PR #38 exists for D3-01** | MEDIUM — D3 work ahead of D2 completion | OK if D3-01 is contract-only (no D2 dependencies). Verify. |
| **Context loss between PKTs** | MEDIUM — each PKT is a separate session potentially | ops/PIPELINE_STATE.md tracks inter-PKT state. Worktree cleanup between PKTs. |

---

## DAG 4: Per-Task TDD Cycle (inside each Implementation task)

```
┌───────────┐     ┌───────────┐     ┌───────────┐     ┌───────────┐     ┌───────────┐
│ Write     │     │ Run test  │     │ Write     │     │ Run test  │     │ Commit    │
│ failing   │────▶│ verify    │────▶│ minimal   │────▶│ verify    │────▶│           │
│ test      │     │ RED       │     │ code      │     │ GREEN     │     │           │
│ ~1 min    │     │ ~30 sec   │     │ ~2 min    │     │ ~30 sec   │     │ ~15 sec   │
└───────────┘     └───────────┘     └───────────┘     └───────────┘     └───────────┘

Total per cycle: ~4-5 min
```

### CONFLICTS in DAG 4:
| Conflict | Risk | Mitigation |
|----------|------|------------|
| **Test passes immediately (false GREEN)** | HIGH — means test doesn't test the right thing | TDD skill: if test passes immediately, test is wrong. Rewrite. |
| **Agent writes code before test** | HIGH — violates TDD iron law | Skill enforcement: tell agent to invoke TDD skill. Validate with TDD checker after. |
| **Test requires DB/external service** | MEDIUM — unit tests should mock, but integration tests need infra | For unit tests: mock. For integration: ensure docker compose is running. |

---

## DAG 5: Review Pipeline (after each PR)

```
           ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
           │ code-reviewer │   │ pr-review-pr │   │ test-analyzer│
           │ (sonnet)      │   │ (sonnet)     │   │ (sonnet)     │
           │ ~3 min        │   │ ~5 min       │   │ ~3 min       │
           │ PARALLEL      │   │ PARALLEL     │   │ PARALLEL     │
           └──────┬────────┘   └──────┬────────┘  └──────┬────────┘
                  │                    │                   │
                  └────────────────────┼───────────────────┘
                                       │
                                       ▼
                    ┌──────────────────────────────────────────┐
                    │ Meta-review: verify reviews are thorough │
                    │ (sonnet, ~2 min)                          │
                    │ BLOCKED BY: all 3 reviewers              │
                    └────────────────────┬─────────────────────┘
                                         │
                              ┌──────────┴──────────┐
                              │                     │
                              ▼                     ▼
                    ┌──────────────┐     ┌──────────────────┐
                    │ PASS → MERGE │     │ FAIL → fix cycle │
                    └──────────────┘     │ dispatch doer    │
                                         │ re-run reviewers │
                                         │ ~10-20 min loop  │
                                         └──────────────────┘
```

### CONFLICTS in DAG 5:
| Conflict | Risk | Mitigation |
|----------|------|------------|
| **3 reviewers return at different times** | LOW — collect all before proceeding | Wait for all 3 + meta-review before acting |
| **Reviewer misses issue (false PASS)** | MEDIUM — meta-reviewer should catch | Meta-reviewer specifically checks: did reviewers cite file:line? Did they check tests? |
| **Fix cycle creates new issues** | MEDIUM — fix for C1 might break something else | Re-run full test suite after every fix, not just the targeted test |

---

## DAG 6: Monitoring (runs continuously alongside all DAGs)

```
        ┌─────────────────────────────────────────────────────┐
        │                 SESSION TIMELINE                     │
        │                                                     │
  :00   │  ┌─Pipeline─┐                                      │
  :03   │  │ Health   │  ┌─Proj11─┐                          │
  :07   │  │ Monitor  │  │ Sync   │                          │
  :11   │  └──────────┘  │Monitor │  ┌─CI/PR──┐              │
  :15   │  ┌─Pipeline─┐  └────────┘  │Monitor │              │
  :22   │  │ Health   │              └────────┘              │
  :30   │  └──────────┘                                      │
  :33   │               ┌─Proj11─┐                           │
  :37   │  ┌─Pipeline─┐ │ Sync   │                          │
  :41   │  │ Health   │ └────────┘  ┌─CI/PR──┐              │
  :45   │  └──────────┘              │Monitor │              │
  :52   │  ┌─Pipeline─┐              └────────┘              │
  :60   │  │ Health   │                                      │
        │  └──────────┘                                      │
        └─────────────────────────────────────────────────────┘
```

### CONFLICTS in DAG 6:
| Conflict | Risk | Mitigation |
|----------|------|------------|
| **Cron fires while agent is mid-response** | ZERO — crons only fire when REPL is idle | Built-in safety |
| **Cron reads ops/ while you're writing it** | LOW — file writes are atomic enough | Negligible risk |
| **Cron reports stale data because agent just committed** | LOW — slight timing mismatch | Cron should re-check after brief delay |
| **Multiple crons fire simultaneously** | LOW — different offsets (:03, :07, :11) | Staggered by design |

---

## DAG 7: Full Project (all waves)

```
┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐
│  D1    │   │  D2    │   │  D3    │   │  D4    │   │  D5    │   │  D6    │   │  G1    │
│  DONE  │──▶│ACTIVE  │──▶│QUEUED  │──▶│QUEUED  │──▶│QUEUED  │──▶│QUEUED  │──▶│QUEUED  │
│  5/5   │   │  1/5   │   │  0/5   │   │  0/5   │   │  0/5   │   │  0/5   │   │  0/5   │
│        │   │ 2.5-4h │   │ 2.5-4h │   │ 2.5-4h │   │ 2.5-4h │   │ 2.5-4h │   │ 2-3h   │
└────────┘   └────────┘   └────────┘   └────────┘   └────────┘   └────────┘   └────────┘

Total estimate: 15-25 hours of agent execution time
Across sessions: ~5-8 sessions (3-4 hours each)
```

### CONFLICTS in DAG 7:
| Conflict | Risk | Mitigation |
|----------|------|------------|
| **D3 work started before D2 done (PR #38)** | MEDIUM — D3-01 is contract-only, may not depend on D2 | Verify D3-01 has no D2 dependencies before allowing it |
| **Wave order violated** | HIGH — skipping waves breaks dependency chain | Enforce: only start wave N+1 after wave N WRAP is Done |
| **Context loss across sessions** | HIGH — multi-day project spanning many sessions | ops/PIPELINE_STATE.md is the recovery mechanism. Commit before every session end. |
| **Build repo issues (159) not aligned with Project 11** | MEDIUM — 159 issues may not map cleanly to 35 Project 11 items | Project 11 is the delivery DAG. Build issues are backlog. Don't let backlog distract from delivery. |
| **Someone else merges to main during a wave** | LOW — Mark is sole developer | If it happens: rebase active worktree before next PKT |
| **G1 (guardrails) should arguably run parallel with D2-D6** | MEDIUM — demo guardrails apply to all waves | G1-03 (harden run timeline) could be done anytime. G1-01/G1-02 need stable surfaces first. Consider interleaving G1-03 with D3-D4. |

---

## CRITICAL CONFLICT SUMMARY

### Must-Prevent (will cause data loss or broken state)

| # | Conflict | Where | Prevention |
|---|----------|-------|------------|
| 1 | Two agents commit to same branch simultaneously | DAG 1, DAG 2 | **NEVER parallel implementers on same branch** |
| 2 | Two agents edit same file | DAG 2, DAG 3 | File ownership in spec. Overlap → merge into one task. |
| 3 | Start D2-02 before PR #166 merged | DAG 3 | D2-02 branches from main. PR #166 must be in main first. |
| 4 | Start new PKT before prior PKT merged to main | DAG 3 | Strictly sequential PKTs within a wave. |
| 5 | Duplicate work (PR #37 exists for D2-01) | DAG 2 | **CHECK FOR EXISTING PRs BEFORE CREATING WORK.** |
| 6 | Session dies with uncommitted ops state | DAG 7 | Commit ops/ after every significant action. |

### Acceptable (managed by protocol)

| # | Conflict | Where | Management |
|---|----------|-------|------------|
| 7 | Test passes immediately (false GREEN) | DAG 4 | TDD skill catches this |
| 8 | Reviewer misses issue | DAG 5 | Meta-reviewer catches this |
| 9 | Fix cycle creates new issues | DAG 5 | Full test suite re-run after every fix |
| 10 | Context loss across sessions | DAG 7 | ops/ state + memories |
| 11 | Agent writes code before test | DAG 4 | TDD skill + TDD checker validation |
| 12 | Rebase conflicts on long-running branches | DAG 3 | Rebase before each PR, small worktree lifespan |
