# Program Plan — NYQST DocuIntelli Delivery

## Strategy

### Mission
Ship NYQST DocuIntelli to demo-ready. This means a buyer can see the platform,
understand its value, and believe it's real — not a prototype.

### Success Criteria
- All 35 Project 11 items Done
- Every PR merged passes full test suite
- No critical or important review issues open at merge
- Every feature traceable: issue → spec → PR → tests → merge
- Demo path works end-to-end (seed → navigate → research → analyze)

### Priority Stack (what matters most, in order)
1. **Unblock D2** — PR #166 has 2 criticals blocking the wave. Fix and merge.
2. **D2 wave delivery** — 4 remaining items (D2-01 through D2-WRAP)
3. **Issue triage** — 189 open issues need to be mapped to deliverables or deferred
4. **Stale PR triage** — 6 open PRs across repos, some stale
5. **Quality infrastructure** — shell API has zero tests (427 lines untested)
6. **Foundation gaps** — duplicate code (formatCost ×3, token aggregation ×4)

### What I'm NOT Doing (scope boundaries)
- Not writing product strategy (Mark's domain)
- Not making architecture decisions without specs from advisor agents
- Not building the org control plane (stub repo, future work)
- Not managing infrastructure (Docker, cloud — separate concern)

---

## Tactics

### How I Execute
1. **Pipelined waves** — SPEC ahead, IMPLEMENT in sequence, REVIEW in parallel, MERGE one at a time
2. **Fractal hierarchy** — I'm director. Lieutenants (opus) manage waves. Doers (sonnet) implement.
3. **Confidence-gated scaling** — prove each level works before adding more agents
4. **Front-loading** — read all issues in wave, write all specs, THEN dispatch implementation
5. **Continuous monitoring** — 3 cron monitors, background prep agents
6. **Issue-traced** — every piece of work has a GitHub issue number
7. **Agent-dispatched** — I never code, review, or test. I dispatch, verify, and merge.

### How I Track
- `ops/PIPELINE_STATE.md` — what's happening NOW (live dashboard)
- `ops/RUN_LOG.md` — what happened (audit trail)
- `ops/AGENT_LEDGER.md` — who can do what (capability map)
- `ops/PROGRAM_PLAN.md` — this file (strategy + tactics + playbooks)
- GitHub Project 11 — delivery order (source of truth)
- GitHub Issues — work items (source of truth)
- TaskList with blockedBy — DAG enforcement for current work

### How I Brief Mark
Status update format (used after every significant milestone):
```
[date] Pipeline: X/35 done
Wave D2: [status]
In flight: [what's running]
Completed since last brief: [what shipped]
Blockers: [any]
Next: [what I'm doing, not what I could do]
```

---

## Playbooks

### Playbook 1: Starting a New Session
1. Read ops/PIPELINE_STATE.md
2. Read ops/RUN_LOG.md (recent)
3. Read memories
4. Check live git state + Project 11
5. Detect drift → fix PIPELINE_STATE.md
6. Start 3 cron monitors
7. Brief Mark on status
8. Execute next action from PIPELINE_STATE.md

### Playbook 2: Implementing a PKT
1. Check for existing PRs (don't duplicate)
2. Read the YOLO issue body + acceptance criteria
3. Read referenced docs only
4. Dispatch advisor to write spec (or front-load it yourself)
5. Spec review loop (advisor + reviewer agent)
6. Create worktree from correct base branch
7. Dispatch doer with SUBAGENT_TICKET format + TDD skill
8. Verify tests pass
9. Draft PR
10. Dispatch 3 reviewers in parallel + meta-reviewer
11. Fix findings, re-verify
12. Merge when all gates pass
13. Dispatch updater to close issues + update board

### Playbook 3: Fixing Review Findings
1. Create GitHub issue for each finding (C1→issue, C2→issue, etc.)
2. DAG: fixes sequential on same branch, verify after all, review, merge
3. Dispatch coding agent for each fix (one at a time on same branch)
4. After all fixes: run full test suite
5. Dispatch reviewer to confirm fixes
6. If reviewer finds new issues: loop

### Playbook 4: Triaging Open Issues
1. Pull all open issues: `gh issue list --limit 500`
2. Categorize: which feed into current wave? Which are backlog?
3. Label/milestone issues that map to Project 11 items
4. Close duplicates
5. Defer issues not relevant to current or next wave
6. Update PIPELINE_STATE.md with triage results

### Playbook 5: Triaging Stale PRs
1. List open PRs: `gh pr list --state open`
2. For each stale (>3 days no update):
   - Is it relevant to current wave? → Review and advance
   - Is it superseded? → Close with comment
   - Is it blocked? → Identify blocker, create issue
3. Update PIPELINE_STATE.md

### Playbook 6: Starting a New Wave
1. Verify previous wave's WRAP is Done on Project 11
2. Read all issues for the new wave (CAP + 3 PKTs + WRAP)
3. Front-load: dispatch spec agents for all 3 PKTs in parallel
4. While specs being written, triage backlog for related issues
5. When specs pass review, begin PKT-01 implementation
6. Brief Mark: "Starting wave [N], expected [time], here's what's in scope"

### Playbook 7: Scaling to Lieutenants
1. Current level proven? (e.g., PR #166 fix cycle completed successfully)
2. Identify scope for lieutenant (e.g., "manage D2 wave")
3. Write TRACK_BRIEF using agentops-template format
4. Dispatch: `claude -p --model opus "[brief]"`
5. Lieutenant reads CLAUDE.md, follows same protocols
6. Monitor via ops/ file updates
7. Intervene only on escalations

### Playbook 8: Handling a Blocked Agent
1. Check agent's reported status (DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED)
2. If NEEDS_CONTEXT: provide missing context, re-dispatch same model
3. If BLOCKED: assess whether it's a task problem or capability problem
4. Task problem: break task smaller, re-dispatch
5. Capability problem: escalate to more capable model
6. Still blocked after escalation: create issue, escalate to Mark

### Playbook 9: Recovering from Session Death
1. Read ops/PIPELINE_STATE.md (last committed state)
2. Read TaskList (pending/in_progress/completed)
3. Check git status in all repos (uncommitted work?)
4. Check for abandoned worktrees: `git worktree list` in Build repo
5. Detect what was in-flight when session died
6. Re-dispatch from last checkpoint

### Playbook 10: Managing Cost
1. Track agent dispatches in RUN_LOG.md (model, duration)
2. Use sonnet for doers/checkers, opus for orchestrators only
3. Haiku only for bulk file reads
4. Front-load reads before dispatching (minimize re-reads)
5. Never idle — prep next task while current runs
6. Monitor: if a task takes >2x expected time, investigate

---

## Operating Model — What's Running At Any Moment

### Standing Team
| Role | Model | Dispatch | Purpose |
|------|-------|----------|---------|
| **Director** (me) | opus | This session | Decisions, approvals, escalations, briefings |
| **Build Lieutenant** | opus | `claude -p --model opus` | Manages current PKT delivery cycle with its own sonnet pod |
| **Prep Lieutenant** | opus | `claude -p --model opus` | Front-loads specs for next 2-3 PKTs in parallel |
| **Ops Agent** | sonnet | Background subagent | GitHub issue updates, board sync, PR comments, triage |
| **Triage Agent** | sonnet | Background subagent (periodic) | Maps 189 issues to deliverables, labels, deduplicates |
| **Monitors** | sonnet | 3 crons | Pipeline health, Project 11 sync, CI/PR status |

### What I Do vs What Delegates Do
| Activity | Who | Why |
|---|---|---|
| Decide what to build next | Me | Strategy, judgment |
| Approve specs before implementation | Me | Quality gate |
| Handle escalations from lieutenants | Me | Judgment under uncertainty |
| Brief Mark | Me | Accountability |
| Adjust priorities based on risk | Me | Program management |
| Write code | Build LT's doer pod | I never code |
| Review code | Build LT's reviewer pod | I never review |
| Update GitHub issues/board | Ops agent | Bookkeeping |
| Triage 189 open issues | Triage agent | Bulk analysis |
| Front-load specs | Prep lieutenant | Judgment but not my bottleneck |
| Run/verify tests | Build LT's pod | Mechanical verification |
| Monitor drift | Crons | Automated, periodic |

### Communication Flow
```
Lieutenants → write to ops/ files → I read aggregate state
Ops agent → updates GitHub → cron alerts me to changes
Monitors → silent when healthy, alert on problems only
Me → dispatch lieutenants via TRACK_BRIEF
Me → dispatch ops/triage via SUBAGENT_TICKET
```

### Typical Session Snapshot
```
ACTIVE:
  [BUILD LT]  PKT D2-01 task 2/3 implementing    ← sonnet doer in worktree
  [PREP LT]   D2-02 spec being written            ← reading issue #8
  [OPS]       Closing issue #167 after C1 merge    ← background sonnet
  [CRON×3]    Monitoring silently                  ← every 15-30 min
  [ME]        Reviewing D2-03 issue body           ← front-loading while waiting
```

### Scaling Stages
| Stage | What's running | Prove before advancing |
|-------|---------------|----------------------|
| **1. Solo** | Me + direct sonnet subagents | PR #166 fix→merge cycle works |
| **2. One lieutenant** | Me + Build LT (opus) + sonnet pod | D2-01 delivered through full pipeline |
| **3. Two lieutenants** | Me + Build LT + Prep LT | Parallel wave work without conflicts |
| **4. Full team** | Me + 2 LTs + Ops + Triage + Monitors | D2 wave completes on time |
| **5. Wave-per-lieutenant** | Me + N lieutenants (one per wave) | Only if D2 proves the model |

**Current stage: 1 (Solo).** Must prove PR #166 cycle before scaling.

## Current Focus Areas

### Immediate (this session)
- [ ] Fix PR #166 criticals (C1, C2) → merge → unblock D2
- [ ] Triage 6 open PRs (which are relevant, which are stale?)

### Short-term (next 2-3 sessions)
- [ ] D2-01: review existing YOLO PR #37, advance or redo
- [ ] D2-02 + D2-03: front-load specs while D2-01 implements
- [ ] D2-WRAP: polish pass after all PKTs land

### Medium-term (next 5-8 sessions)
- [ ] D3 through D6 waves (20 items)
- [ ] G1 guardrails (interleave where possible)
- [ ] Issue triage: map 189 Build issues to deliverables

### Deferred
- [ ] Build the org control plane (nyqst-org-control)
- [ ] Infrastructure / Oracle Cloud deployment
- [ ] PropSygnal / RegSygnal domain modules
